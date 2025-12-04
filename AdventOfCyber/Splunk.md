Total event count over time, grouped by day, to determine the number of events captured per day -> ``index=main sourcetype=web_traffic | timechart span=1d count`` -> This helps identify the day that received an abnormal number of logs.

Append the `reverse` function at the end to display the results in descending order, showing the day with the maximum number of events at the beginning. -> ``index=main sourcetype=web_traffic | timechart span=1d count | sort by count | reverse``

`user_agent` field is what tool has been used to do the request. (eg. Chrome, curl, wget)
`client_ip` field contains the IP adresses of the clients accessing the web server. It may help spot IPs with abnormal amount of requests.
`path` contains the URI being requested and accessed by the client IPs.

To filter out some values use `!=`. Eg. ``index=main sourcetype=web_traffic user_agent!=*Mozilla* user_agent!=*Chrome* user_agent!=*Safari* user_agent!=*Firefox*``

To get the IP adresses that do not send requests from common desktop or mobile browsers, use query: ``sourcetype=web_traffic user_agent!=*Mozilla* user_agent!=*Chrome* user_agent!=*Safari* user_agent!=*Firefox* | stats count by client_ip | sort -count | head 5``. The results will be sorted by count in reverse order. (due to `-` in the `sort -count`. It is the same as using the reverse function)

To search in the initial probing of exposed configuration files use the query: ``sourcetype=web_traffic client_ip="<REDACTED>" AND path IN ("/.env", "/*phpinfo*", "/.git*") | table _time, path, user_agent, status``

Search for common path traversal and open redirect vulnerabilities: ``sourcetype=web_traffic client_ip="<REDACTED>" AND path="*..*" OR path="*redirect*"``

To look for footprints of path traversal attempts (`../../`): ``sourcetype=web_traffic client_ip="<REDACTED>" AND path="*..\/..\/*" OR path="*redirect*" | stats count by path``

Check for SQL Injection Attack: ``sourcetype=web_traffic client_ip="<REDACTED>" AND user_agent IN ("*sqlmap*", "*Havij*") | table _time, path, status``

Search for attempts to download large, sensitive files (backups, logs): ``sourcetype=web_traffic client_ip="<REDACTED>" AND path IN ("*backup.zip*", "*logs.tar.gz*") | table _time path, user_agent``

Requests for sensitive archives like `/logs.tar.gz` and `/config` indicate the attacker is gathering data for double-extortion: ``sourcetype=web_traffic client_ip="<REDACTED>" AND path IN ("*bunnylock.bin*", "*shell.php?cmd=*") | table _time, path, user_agent, status`` This attack is called Remote code Execution (RCE) and let's you run commands on the server.

Check if firewall was allowed from the compromised server IP as source and the attacker IP as the destination: ``sourcetype=firewall_logs src_ip="10.10.1.5" AND dest_ip="<REDACTED>" AND action="ALLOWED" | table _time, action, protocol, src_ip, dest_ip, dest_port, reason``

Calculate the sum of the bytes transferred, using the bytes_transferred field: ``sourcetype=firewall_logs src_ip="10.10.1.5" AND dest_ip="<REDACTED>" AND action="ALLOWED" | stats sum(bytes_transferred) by src_ip``

**Conclusions:**
- **Identity found:** The attacker was identified via the highest volume of malicious web traffic originating from the external IP.
- **Intrusion vector:** The attack followed a clear progression in the web logs (`sourcetype=web_traffic`).
- **Reconnaissance:** Probes were initiated via cURL/Wget, looking for configuration files (`/.env`) and testing path traversal vulnerabilities.
- **Exploitation:** The use of `SQLmap` user agents and specific payloads (`SLEEP(5)`) confirmed the successful exploitation phase.
- **Payload delivery:** The Action on Objective was established by the final successful execution of the command `cmd=./bunnylock.bin` via the webshell.
- **C2 confirmation:** The pivot to the firewall logs (`sourcetype=firewall_logs`) proved the post-exploitation activity. The internal, compromised server (`SRC_IP: 10.10.1.5`) established an outbound C2 connection to the attacker's IP.