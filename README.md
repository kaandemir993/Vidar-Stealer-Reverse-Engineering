# Vidar Stealer 2.0 Reverse Engineering Analysis

## Overview

This analysis covers an in-depth reverse engineering of **Vidar Stealer 2.0**, a sophisticated information stealer that uses advanced evasion techniques to remain undetected. Key findings include:

- **Task Scheduler Tampering:** Modifies system task timestamps to 1999 to evade forensic analysis.
- **Explorer.exe Process Hollowing:** Injects shellcode into the legitimate Windows Explorer process.
- **Microsoft Copilot Injection:** Targets the new Microsoft Copilot process for stealthy execution.
- **Persistence:** Maintains presence through scheduled tasks and system process hijacking.


## 1. Task Scheduler Tampering (1999 Timestamps)

One of the most unusual and sophisticated behaviors observed in Vidar Stealer 2.0 is its interaction with the Windows Task Scheduler. The malware does not simply create a new task for persistence—it actively tampers with the timestamps of existing system tasks, setting them to the year **1990**. 

### Why is this significant?

- **Anti-Forensic Technique:** Changing timestamps to an old date (like 1990) disrupts forensic analysis and log reviews, making it harder to determine when the malware first appeared.
- **Evasion:** By altering the timestamps of legitimate system tasks, Vidar blends in, making its own scheduled task less suspicious.
- **Persistence:** Once its own task is created, Vidar can revert the timestamp back to the current date to avoid further suspicion while keeping a foothold on the system.

**Key Observations:**

- System tasks like `OneDrive` had their timestamps changed to `6.04.2026` but some were reverted.
- A custom task created by Vidar was set to `30.11.1999 00:00:00` (a classic anti-forensic timestamp).
- Even after removal, some timestamp changes remained, indicating deep system-level manipulation.

![OneDrive Task Timestamp](images/onedrive_gorev.jpeg#width=200px)


## 2. Explorer.exe – Process Hollowing & Shellcode Injection

One of the most critical evasion techniques used by Vidar Stealer 2.0 is **Process Hollowing** on `Explorer.exe`. This technique allows the malware to execute its malicious code inside a legitimate Windows process, making detection significantly harder.

### How It Works:

- **Target Process:** Vidar targets `Explorer.exe`, a trusted system process.
- **Hollowing:** The malware creates a suspended instance of Explorer and replaces its memory with malicious shellcode.
- **Execution:** The shellcode is then executed within the context of Explorer, bypassing many security controls.

### WinDbg Analysis:

Using WinDbg, we can observe the injected shellcode inside the Explorer process. The memory regions show typical `PAGE_EXECUTE_READWRITE` permissions, which are uncommon for legitimate Explorer memory.

**Key Observations:**

- **Memory Regions:** Suspicious `PAGE_EXECUTE_READWRITE` sections were found in Explorer's address space.
- **Shellcode Patterns:** The injected code contains typical payload structures, including API hashing and anti-debug routines.
- **Persistence:** The shellcode ensures Vidar remains active even after system reboots.

**What the Image Shows:**

The image below is a direct WinDbg dump of the shellcode found inside the `PAGE_EXECUTE_READWRITE` region of Explorer.exe. The disassembly reveals typical shellcode characteristics: prologue instructions (`push`, `mov`), API call patterns (`call`, `jmp`), and obfuscated data blocks.

<img src="images/explorer_shellcode.jpeg" alt="Explorer Shellcode" width="400">


## 3. Microsoft Copilot – A New Target for Process Injection

One of the most remarkable findings in this analysis is the injection of Vidar's shellcode into the **Microsoft Copilot** process. This is a relatively new and highly interesting target, as Copilot is an AI-powered assistant integrated into modern Windows systems.

### How It Works:

- **Target Process:** Vidar targets `Copilot.exe`, a new and trusted Windows process.
- **Injection Method:** The malware uses similar process injection techniques (likely APC injection or thread hijacking) to execute its shellcode within Copilot.
- **Stealth:** Since Copilot is a legitimate process, the shellcode can run undetected by many traditional security solutions.

### WinDbg Analysis – Obfuscated Payload

Initial analysis of the injected code revealed an **obfuscated payload**, with instructions such as `jg`, `add`, and `xchg` appearing out of context. This is a clear indication of code obfuscation, used to evade static analysis.

![Copilot Injection - Obfuscated Payload](images/copilot_obfuscated.jpeg#width=200px)

### WinDbg Analysis – API Calls & Obfuscation

The image below shows a mix of clear API calls and obfuscated code. The resolved functions include:

- `LoadLibraryExW` – Used to load additional DLLs.
- `LoadLibraryW` – Standard DLL loading.
- `LocalFree` – Memory management.
- `MultiByteToWideChar` – String conversion (commonly used in malware for data exfiltration).

The unresolved sections (`???`, `jg add`, etc.) indicate areas where Windbg could not fully decode the obfuscated code, highlighting the complexity of Vidar's evasion techniques.

![Copilot API Calls and Obfuscated Code](images/copilot_api_obfuscated.jpeg#width=200px)


## Conclusion

Vidar Stealer 2.0 is a prime example of how modern information stealers have evolved to become highly sophisticated and resilient. This analysis uncovered several advanced techniques that set it apart from typical malware:

- **Task Scheduler Tampering (1990 Timestamps):** A clever anti-forensic technique that disrupts log analysis and hides persistence mechanisms.
- **Explorer.exe Process Hollowing:** Demonstrates Vidar's ability to execute malicious code within a trusted system process, bypassing many security controls.
- **Microsoft Copilot Injection:** Targets a new and emerging attack surface, showing how threat actors are adapting to new Windows features.
- **Obfuscated API Calls:** The use of obfuscation and dynamic API resolution makes static analysis challenging and highlights the malware's evasion capabilities.

### Key Takeaways

- **Persistence:** Vidar maintains persistence through scheduled tasks and system process injection.
- **Evasion:** Anti-forensic timestamps, obfuscation, and trusted process injection make detection difficult.
- **Data Theft:** The use of `LoadLibrary` and `MultiByteToWideChar` confirms its ability to load additional modules and exfiltrate data.

### Sample Download

The analyzed sample is available on MalwareBazaar for those who wish to conduct their own analysis:

🔗 **[Vidar Stealer 2.0 Sample on MalwareBazaar]( https://bazaar.abuse.ch/sample/8a094a17335433c16502a1e9d650dea5bd8eb8e56c7fae884b0f0302dccba482)**

### Final Thoughts

Vidar Stealer 2.0 represents a new generation of information stealers that are not only focused on data theft but also on staying undetected. Its ability to tamper with system tasks, inject into trusted processes, and target new Windows features like Copilot demonstrates a deep understanding of modern security controls.

Understanding these techniques is crucial for defenders to build better detection rules and security measures against evolving threats.

