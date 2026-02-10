Another name is "ILOVEYOU" worm, released in 2000. It was distributed via email and had successfully infected five millions computes.
This malware has 2 ways of duplication:
- Sending itself as an attached `.txt` file to all recipients in MS Outlook address book.
- Sending through phone line as a `.htm` file for all users that were connected to the same IRC channel (during that time, internet and phone line are the same).
Why is was a successful attack?
- Windows Scripting Hosts automatically allowed all scripts to be executed.
- In Windows file extension can be hidden. So that `.txt` file looks like a general file.
- People in general trust their friends (in address book).
- The scripts run with user rights in the system.
- Also social engineering aspect of this attack is truly effective
The procedure of this attack:
- Creates files in the file systems: system directory, window directory.
- Manipulates the windows registry to guarantee execution at windows startup.
- Tries to download a password stealing trojan horse.
- Tries to send itself to all people in the address book.
- Overwrite script files.
- Overwrites IRC script files.
Analysis:
- VBS
- Registry information within Windows operating system.
- MS Outlook API.