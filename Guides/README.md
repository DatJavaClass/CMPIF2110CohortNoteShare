<p align="center"><img width="511" height="768" alt="PatchLad" src="https://github.com/user-attachments/assets/70a8d516-5a21-4829-a594-b7b6222028ae" /></p>

# Guides

Standalone walkthroughs and setup guides for the cohort, gathered here so they live in one place. Each guide is its own file. Read whichever one you need.

## Getting connected

**1. [AltConnect: Pitt VPN Without the GlobalProtect Client](AltConnectInstructions.md)**
The unofficial way onto Pitt's VPN: same VPN, same credentials, nothing from Palo Alto installed on Windows. It runs OpenConnect inside WSL2, so the tunnel lives entirely in Linux and Windows never meets the client. Five short steps: get WSL if you lack it, install one small package, run the connect command and approve the Duo push, work from a second WSL tab (with VS Code riding the tunnel through its WSL badge), and `Ctrl+C` to disconnect clean. Also covers the one trick Access users need, a small relay so Microsoft Access on the Windows side can still reach the course database server, plus fixes for the two things that go sideways (WSL's DNS relay, the hourly drop) and a one-line removal when you are done with it for good.

## SQL reference

**2. [SQL Statements Field Guide](CMPIF2110_SQL_Statements_Field_Guide.pdf)**
A four-page reference covering basic, intermediate, and advanced SQL statements, followed by the operators and functions that turn those statements into useful queries. It begins with the SQL introduced in Module 3 Part 1, then adds the common tools most likely to appear later in CMPIF2110 or everyday database work. Every entry includes a one-sentence description and an example using MySQL 8.0 syntax where database systems differ.

---

*Browsing note: The VPN guide is plain Markdown you can read directly on GitHub. The SQL field guide is a PDF that opens in GitHub's document viewer.*
