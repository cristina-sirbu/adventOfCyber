Reveal hidden files => ``ls -la``
Search string in file => ``grep "Failed password" auth.log``
Check where you are in the filepath => ``pwd``
Which user is logged => ``whoami``
Check history of commands ran => ``history``

Scan for open ports => ``nmap -``
Check all ports and what runs behind them => ``nmap -p- --script=banner 10.81.184.119``

Get all UDP ports => ``nmap -sU 10.81.184.119``
 Perform advanced DNS queries => ``dig @10.81.184.119 TXT key3.tbfc.local +short``

 Check list open ports from inside the machine => ``ss -tunlp``

