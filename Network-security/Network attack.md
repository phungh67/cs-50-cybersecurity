
One of the famous attack is IP spoofing - try to pretend that you are the legitimate client to make request to destination server with a purpose to deny the true request.

But forging the fake packet is quite hard:
- Must know the destination and source - in fact you must successfully capture a packet in-transit to have some of mandatory ingredients.
- But even that, you need to know the sequence number and many more things to successfully attack.
- Not to mention about the firewall and various RFC things

Nevertheless, with this type of attack still be a huge threat:
- Anonymity of attacker
- Mass attacks
- Many layers are still vulnerable

# 1. Reconnaissance

To be successfully conduct an attack (and of course, gaining some valuable information), choosing what to attack and how to attack are also critical questions.

One of famous tool for this is `nmap` - comes pre-installed in `kali-linux`. To check the official page about this tool, refer to [nmap-guide-basic](https://nmap.org/book/man-briefoptions.html)

Once the scan phase is finished, the forging phase can be started. But before that, to gain a necessary information: source, destination, sequence number, ... another tool is needed `wireshark`.

These two tools are very powerful and lie on the arsenal of `kali-linux` as well as appear in the [[Lab 1 - Nmap and WireShark]]
