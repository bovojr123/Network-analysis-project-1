![IMG_8901](https://github.com/user-attachments/assets/bd6a44ac-6823-416d-b3c3-1a8ca78d0e01)
# Network-analysis-project-1
Network analysis report: PCAP Using Wireshark
On 2025-02-24, I analyzed a 15 minute PCAP (17:02:19–17:17:22). The highest volume of traffic to the victim (10.4.1.101) originated from 94.103.84.245. Following HTTP streams revealed an HTTP GET that delivered a file hosted on foodsgoodforliver.com. The file was submitted to VirusTotal and returned a detection score of 58/72. The captured HTTP stream also shows an intermediary indicating mitmproxy behavior. Findings suggest a suspicious file download to the victim host; further malware analysis is recommended.


Observations & methodology
	•	Time window analyzed: 2020-04-01, 17:02:19 — 17:17:22 (15 minutes).
	•	Primary tools: Wireshark for interactive analysis and stream decoding; follow-up sandbox/VirusTotal checks.
	•	Initial triage steps
	1.	Calculated top talkers by byte count to find hosts with the largest data transfer to destination 10.4.1.101.
	2.	Focused on the highest byte source, 94.103.84.245.
	3.	Applied display filters to isolate HTTP activity and followed HTTP streams to recover request/response content.
	4.	Searched for other HTTP sessions (http) to identify additional downloads.
	5.	Used NetBIOS Name Service filters (nbns) to identify the local victim hostname.
	6.	Retrieved the downloaded file and uploaded it to VirusTotal for a quick verdict.

⸻![IMG_8906](https://github.com/user-attachments/assets/38545760-af8f-46df-b6be-28cae82acf01)
![IMG_8908](https://github.com/user-attachments/assets/b314ebe5-02b8-4920-bbc9-6d2a98298899)
![IMG_8903](https://github.com/user-attachments/assets/2d764420-b831-4fe0-8e23-2b09da73e440)


Key findings 
	•	Top talker: 94.103.84.245 → 10.4.1.101 (largest byte count during the capture window).
	•	Wireshark filter used to locate requests from that host: ip.addr == 94.103.84.245 && http.request
(This returns HTTP request packets where either source or destination IP is 94.103.84.245.)

HTTP evidence: Followed the HTTP stream for the suspicious TCP stream and found an HTTP GET/POST to /docs.php (the decoded stream shows a request for docs.php and a binary-like response body).
	•	Multiple HTTP sessions found: A global filter http showed several HTTP packets; individual streams were reviewed via “Follow → HTTP Stream” to decode payloads.
	•	Server/intermediary observed: The HTTP response contains Server: mitmproxy 6.0.0.dev, and the client view includes an HTTP response HTTP/1.1 502 Bad Gateway in the captured stream — indicating a proxy/interception artifact in the capture (or the upstream server returned 502).
	•	Downloaded file origin: Hostname referenced in captured requests corresponds to foodsgoodforliver.com (the domain that served the file).
	•	Malware quick check: The downloaded artifact was submitted to VirusTotal and returned 58/72 detections (high risk).
	•	Victim identification: Using the nbns display filter (nbns) allowed recovery of the NetBIOS/hostname information for the victim machine that received the file.
	![IMG_8905](https://github.com/user-attachments/assets/edc1b53a-5517-46bc-854f-ebdec8f61da3)
![IMG_8914](https://github.com/user-attachments/assets/ec431b3a-eb97-4620-8d2a-61cb31590315)
![IMG_8913](https://github.com/user-attachments/assets/4720f9e4-4296-4ea4-a797-36b71f7b423d)
![IMG_8911](https://github.com/user-attachments/assets/3e2514cf-a2e3-44ad-9190-03b6f45f8fa4)
![IMG_8909](https://github.com/user-attachments/assets/4d4aa61e-52be-43c7-bfd7-8c93fef9ef37)
![IMG_8907](https://github.com/user-attachments/assets/e368a6d9-d0e1-4ed2-ae5d-91169b4a1155)

Conclusion & recommendations
	•	The capture shows a suspicious file transfer to 10.4.1.101 originating from 94.103.84.245 and associated with the domain foodsgoodforliver.com. VirusTotal results (58/72) indicate the file is likely malicious.
	•	Immediate actions
	•	Isolate the host 10.4.1.101 for containment.
	•	Preserve the original PCAP and extracted file (hash them: SHA256/SHA1/MD5) and add them to forensic evidence storage.
	•	Submit the file to a controlled sandbox for behavioral analysis (if not already done).
	•	Block 94.103.84.245 and the offending domain at network perimeter/DNS and add indicators to IDS/IPS rules.

  
