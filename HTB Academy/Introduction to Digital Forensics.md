# Introduction to Digital Forensics

## Forensics Imaging

- FTK Imager
- AFF4 Imager
- dd
- Virtualization Tools

Mount ->  [Arsenal Image Mounter](https://arsenalrecon.com/downloads)

## Extracting Host-based Evidence & Rapid Triage

#### Host-based Evidence

Capture volatile memory, bisa pakai FTK Imager, some other:
- [WinPmem](https://github.com/Velocidex/WinPmem): WinPmem has been the default open source memory acquisition driver for windows for a long time. It used to live in the Rekall project, but has recently been separated into its own repository.
- [DumpIt](https://www.magnetforensics.com/resources/magnet-dumpit-for-windows/): A simplistic utility that generates a physical memory dump of Windows and Linux machines. On Windows, it concatenates 32-bit and 64-bit system physical memory into a single output file, making it extremely easy to use.
- [MemDump](http://www.nirsoft.net/utils/nircmd.html): MemDump is a free, straightforward command-line utility that enables us to capture the contents of a system's RAM. It’s quite beneficial in forensics investigations or when analyzing a system for malicious activity. Its simplicity and ease of use make it a popular choice for memory acquisition.
- [Belkasoft RAM Capturer](https://belkasoft.com/ram-capturer): This is another powerful tool we can use for memory acquisition, provided free of charge by Belkasoft. It can capture the RAM of a running Windows computer, even if there's active anti-debugging or anti-dumping protection. This makes it a highly effective tool for extracting as much data as possible during a live forensics investigation.
- [Magnet RAM Capture](https://www.magnetforensics.com/resources/magnet-ram-capture/): Developed by Magnet Forensics, this tool provides a free and simple way to capture the volatile memory of a system.
- [LiME (Linux Memory Extractor)](https://github.com/504ensicsLabs/LiME): LiME is a Loadable Kernel Module (LKM) which allows the acquisition of volatile memory. LiME is unique in that it's designed to be transparent to the target system, evading many common anti-forensic measures.

WinPmem
```
winpmem_mini_x64_rc2.exe memdump.raw
```

#### Rapid Triage

This approach emphasizes collecting data from potentially compromised systems. The goal is to centralize high-value data, streamlining its indexing and analysis.

Tool: [KAPE (Kroll Artifact Parser and Extractor)](https://www.kroll.com/en/services/cyber-risk/incident-response-litigation-support/kroll-artifact-parser-extractor-kape).

What if we wanted to perform artifact collection remotely and en masse? This is where EDR solutions and [Velociraptor](https://github.com/Velocidex/velociraptor) come into play. Endpoint Detection and Response (EDR) platforms offer a significant advantage for incident response analysts. [Velociraptor](https://github.com/Velocidex/velociraptor) is a potent tool for gathering host-based information using Velociraptor Query Language (VQL) queries. Beyond this, Velociraptor can execute `Hunts` to amass various artifacts.

