# Computer Forensics - Spring 2023

## Assignment 1: enron_search

#### An Email Forensic Analysis Tool

## Team members

| Name                         | TAMUCC ID | Email                              |
|------------------------------|-----------|------------------------------------|
| `Venkata Sai Pavan Arepalli` | A04268973 | varepalli@islander.tamucc.edu      |
| `Siva Parvathi Chirumamilla` | A04269507 | schirumamilla2@islander.tamucc.edu |
| `Ganesh pulyala`             | A04267491 | gpulyala@islander.tamucc.edu       |

&nbsp;

## For this assigment we have four tasks:

- Data set preparation
- Tool development
- Execution and testing
- <span style="background-color:green;color:white;padding:3px">Optimization</span>

### 1. Data set preparation

- Download the Enron email data set folder from the [link](https://www.cs.cmu.edu/~enron/).
- Follow the instructions in the [link](https://github.com/lintool/Enron2mbox) to convert the data set from maildir to
  mBox format.

### 2. Tool development

* We used java to develop the tool.
* We added a total of three commands into the tool.
    * `term_search` Prints the mails with all the given search terms in it.
      * `enron_search term_search [term, term, term...]`
    * `address_search` Prints all the emails sent and received by a given person.
      * <span style="color:green">We also added further support for middle_name for better results.</span> 
        * `enron_search address_search first_name last_name`
        * `enron_search address_search first_name MIDDLE_NAME last_name`
    * `interaction search` Prints all the mails exchanges between two users.
       * `enron_search interaction_search mail_one mail_two`

* Each command has its own working principle explained later.

### 3. Execution and testing

* Pre- requisites
    * Java installed
    * Intellij IDE to run the code. Download from [link](https://www.jetbrains.com/idea/download/#section=mac) if not
      already installed.
* Download the source code.
* Open the project in IDE, and wait for gradle sync.
* Run the project using `Main.java` file.
* In the console enter any of the above listed command and provide valid input.
* Result is Printed in the console.

### <span style="background-color:green;color:white;padding:3px">Optimization</span>

We optimized the complete tool in multiple stages

- `Simple code optimization` Simple code changes to decrease memory and processing time.
    * Examples:
        * `Optimizing loops`
        * `Decreasing object redundancy` by re-using object like stores and sessions.
- `Decrease search scope`
    * Initially we searched all the mail boxes.
    * Cons of this approach is the as per mBox format all the mails belonging to a person are arranged into files with
      his name. So searching through the mail boxes is serious misuse of the processing power, and resources.
    * A typical mailbox of mBox is named as `last_name.[0]first_name._mailbox_name`.
    * Example mailbox name of person named `John Arnold` is `enron.arnold-j._sent_mail` representing the `Sent mailbox`.
    * Decreasing the scope of the search will tremendously optimize the tool.
    * By doing so the radius of the search is decreased from a total of 3455 files to a mere 50-100 files.
    * There are some exceptional scenarios where no files under person name ( maybe accidentally deleted, or the persons
      mail is completely terminated) , making the search radius 0.
    * To handle these scenarios we perform a deep search into all the files.

- <span style="background-color:green;color:white;padding:3px">**Resource optimization using Parallel Processing and
  Threads in java**</span>
    * We used threads to divide the computation work between `multiple threads`.
    * We used all the available in a `Processor cores`.
    * So everytime we run a command the computation is distributed between multiple thread.
    * The number Threads is always equal to the number cores available in the processor. As there is no use to spawn
      more thread and number of CPU cores.

#### Command Algorithms:

* `term_search`
    * Get the list of all the mail boxes.
        * Distribute the list between threads.
            * `Inside each thread`
                * Loop through the mail boxes.
                    * Loop through all the messages in mailbox.
                        * Check if message body has all the search terms
                        * If yes save the Sender and date to Result array.
    * Print the final result and size of result.<br>

* `address_search`
    * Get the list of all the mail boxes `associated to the person`.
        * Distribute the list between threads.
            * `Inside each thread`
                * Loop through the mail boxes.
                    * Loop through all the messages in mailbox.
                        * Check `From`,`To`, `X-FRom`, `X-To` contains the first and last name of the person.
                        * If yes.
                            * If the person is `sender` save the address to result array.
                            * If the person is the `receiver`.
                                * `Process the list of receivers and remove mails of other persons`.
                                * Save the address the result array.
    * Clean the result array and `remove any redundant addresses`.
    * Print the final result and size of result.<br>
* `interaction search`
    * Get the list of all the mail boxes `associated to the both mail addresses given`.
        * Distribute the list between threads.
            * `Inside each thread`
                * Loop through the mail boxes.
                    * Loop through all the messages in mailbox.
                        * Consider `From`,`To`, `X-FRom`, `X-To`
                        * Check if the address of person_1 and person_2 is found in To or From fields.
                        * If yes save the result in the format of`Sender -> Receiver [Subject] Date` to result array.
    * Print the final result and size of result.<br>

### Results of Optimization:

Below is a comparison table of processing time before and after optimization.

| Command              | Initial processing time | Optimized processing time | Percentage                                     |
|----------------------|-------------------------|---------------------------|------------------------------------------------|
| `term_search`        | 26520 ms                | `4107 ms`                 | <span style="color:green">550 % Approx</span>  |
| `address_search`     | 14060 ms                | `198 ms`                  | <span style="color:green">7000 % Approx</span> |
| `interaction_search` | 13890 ms                | `231 ms`                  | <span style="color:green">6000 % Approx</span> |

All the results are in `milliseconds`.

All the command processed on a machine with `Apple Silicon M2 processor with 10 Cores`.
