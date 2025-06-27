1. Prepare Your Environment
bash
# Update Kali
sudo apt update && sudo apt upgrade -y

# Install required tools
sudo apt install john fcrackzip unzip p7zip-full -y

# Unzip rockyou.txt wordlist (if not already done)
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
2. Basic ZIP Cracking with fcrackzip
Dictionary Attack
bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt target.zip
-u: Verify passwords (reduce false positives)

-D: Dictionary mode

-p: Path to wordlist

Brute-Force Numbers
bash
fcrackzip -u -b -c '1' -l 4-6 target.zip
-b: Brute-force

-c '1': Digits only (a=lowercase, A=uppercase)

-l 4-6: Password length range

3. Advanced Cracking with John the Ripper
Step 1: Extract Hash
bash
zip2john target.zip > zip_hash.txt
Step 2: Crack Hash
bash
# Dictionary attack
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

# Brute-force numbers (4-6 digits)
john --incremental=digits --min-length=4 --max-length=6 zip_hash.txt
Step 3: Show Results
bash
john --show zip_hash.txt
4. GPU Acceleration with Hashcat
bash
# Convert to Hashcat format
zip2john target.zip | awk -F: '{print $2}' > hash.hc

# Crack with GPU (mode 13600 for ZIP AES)
hashcat -m 13600 -a 3 hash.hc ?d?d?d?d?d?d  # 6-digit brute-force
hashcat -m 13600 hash.hc /usr/share/wordlists/rockyou.txt  # Wordlist
5. Handling Special Cases
AES-Encrypted ZIPs
Use John/Hashcat (as above) - fcrackzip won’t work.

Multi-Password ZIPs
Each file has a unique password. Use this Python script to automate extraction:

python
import zipfile

def extract_zip(zip_path, password_list):
    with zipfile.ZipFile(zip_path) as zf:
        for file in zf.namelist():
            for pwd in password_list:
                try:
                    zf.extract(file, pwd=pwd.encode())
                    print(f"Extracted {file} with: {pwd}")
                    break
                except:
                    continue

# Usage:
extract_zip("target.zip", ["123456", "password", "111111"])
6. Saving Results
bash
# Save cracked passwords
john --show zip_hash.txt > cracked.txt

# Backup hashes for future testing
cp zip_hash.txt ~/password_cracking_tests/
7. Optional: Create Custom Wordlists
bash
# Common number patterns
crunch 4 6 0123456789 -o numbers.txt

# Combine wordlists
cat /usr/share/wordlists/rockyou.txt numbers.txt > custom_wordlist.txt
Cheat Sheet Table
Task	Command
Dictionary Attack	fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt target.zip
Brute-Force Numbers	fcrackzip -u -b -c '1' -l 4-6 target.zip
Extract Hash	zip2john target.zip > zip_hash.txt
Crack with John	john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
Show Results	john --show zip_hash.txt
GPU Cracking	hashcat -m 13600 hash.hc /usr/share/wordlists/rockyou.txt
Pro Tips
Always test on your own files first.

Use unzip -t to verify ZIP integrity.

For large files, use screen to prevent session timeouts:

bash
screen -S cracking_session
john zip_hash.txt
# Ctrl+A then D to detach
