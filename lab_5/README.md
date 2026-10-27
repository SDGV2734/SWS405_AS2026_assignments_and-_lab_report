# Lab 5

![College of Science and Technology Logo.jpeg](Lab%205/College_of_Science_and_Technology_Logo.jpeg)

$$
\huge \begin{array}{c} \mathbf{\textsf{Royal University of Bhutan,}} \\ \mathbf{\textsf{College of Science and Technology}} \end{array}
$$

$$
\Large \mathbf{\textsf{SWS405: Digital Forensics}}
$$

$$
\Large \mathbf{\textsf{Computing Technologies Department  }}
$$

$$
\large \mathbf{\textsf{Lab Report 5  }}
$$

$$
\large \mathbf{\textsf{Submitted by: Sonam Dorji Ghalley  }}
$$

$$
\large \mathbf{\textsf{Student No: 02230299  }}
$$

## 1. Introduction

Email based social engineering attacks and forensics investigations of compromised hosts are related within security defence. A single user clicking a malicious link or attachment will provide an attacker with their first foothold into the organisations network. The investigator will need to understand what happened on the impacted machine by looking at the artefacts left behind by the OS.

**Lab 1: Phishing Analysis Fundamentals.** This introductory room teaches how an email is structured and delivered, and how to judge whether a message is malicious or benign. It covers the anatomy of an email address, the roles of SMTP, POP3 and IMAP, the contents of email headers and bodies, the reconstruction of attachments, and the main categories of malicious email (spam, phishing, spear phishing, whaling, smishing and vishing). The practical work uses sample messages stored on a virtual machine.

**Lab 2: Windows Forensics 1.** This room introduces registry forensics on Microsoft Windows, the most widely used desktop operating system. The Windows Registry stores configuration data about hardware, software and users, including evidence of recently used files, executed programs and connected devices. The room explains where registry hives are stored on disk, how to acquire them, and how to interpret artifacts such as user accounts, execution history and USB device records. It ends with a hands-on investigation of a triage collection from a workstation at a fictional research laboratory, "Organization X", where unauthorised access is suspected.

## 2. Aim

The aim of these laboratories was to develop practical, entry-level skills in security operations and digital forensics by (a) analysing suspicious email messages to determine their origin, intent and risk, and (b) examining Windows registry artifacts from a triage collection to reconstruct user and system activity, in a controlled environment and without risk to any live system.

## 3. Objectives

**Lab 1: Phishing Analysis Fundamentals**

1. Explain the structure of an email address and the journey of an email from sender to recipient, including the roles of SMTP, POP3, IMAP and DNS.
2. Inspect email headers and the raw message source to identify the sender, subject, date and originating IP address.
3. Examine the email body and its HTML source, and reconstruct a base64-encoded attachment.
4. Classify malicious email types and identify common indicators of a phishing message.
5. Apply safe-handling practice by defanging URLs and IP addresses before recording them.

**Lab 2: Windows Forensics 1**

1. Describe the structure of the Windows Registry, its five root keys and the on-disk locations of the registry hives.
2. Explain why forensic copies of hives, transaction logs and backups are used instead of working on a live registry.
3. Compare registry analysis tools and select appropriate ones for a task.
4. Extract system information, user account data, file and folder usage, evidence of execution and USB device history from registry artifacts.
5. Apply these techniques to the Organization X triage data to answer the investigation questions.

## 4. Tools and Environment

Both labs were completed through the TryHackMe browser interface, using a virtual machine started in split-screen mode. No personal or production systems were used.

**Lab 1: Phishing Analysis Fundamentals** (difficulty: easy; estimated time: 30 minutes)

| Tool / resource | Purpose |
| --- | --- |
| TryHackMe lab virtual machine | Isolated environment holding the `Email Samples` folder on the desktop |
| Mozilla Thunderbird | Opening `email1.eml` and `email3.eml`; viewing headers and the message source (View > Message Source, or Ctrl+U) |
| Text viewer | Reading the source of `email2.txt`, which contains an encoded attachment |
| CyberChef | Decoding base64 to reconstruct the attachment, and defanging URLs and IP addresses |
| Base64-to-PDF converter | Alternative method of rebuilding the PDF attachment |

**Lab 2: Windows Forensics 1** (difficulty: medium; estimated time: 60 minutes)

| Tool / resource | Purpose |
| --- | --- |
| TryHackMe Windows virtual machine | Analysis workstation; the Desktop holds the `triage` and `EZtools` folders |
| `triage` folder | KAPE-collected evidence set with the same directory structure as the source system |
| Registry Explorer (Eric Zimmerman) | Loading multiple hives, merging transaction logs into "clean" hives, and browsing bookmarked forensic keys |
| AppCompatCache Parser (Eric Zimmerman) | Parsing the ShimCache from the SYSTEM hive into CSV |
| EZViewer (Eric Zimmerman) | Viewing the CSV output |
| ShellBag Explorer (Eric Zimmerman) | Parsing ShellBags from user hives |

The room also describes KAPE, Autopsy, FTK Imager, AccessData Registry Viewer and RegRipper. These are discussed for data acquisition and comparison, but the practical analysis used the Eric Zimmerman tools listed above.

## **5. Procedure**

### **5.1 Lab 1: Phishing Analysis Fundamentals**

#### Task 1

1. **Environment set-up.** The lab machine was started in split-screen mode, and the `Email Samples` folder on the desktop was located.
    
    ![Screenshot 2569-10-28 at 12.40.24 PM.png](Lab%205/Screenshot_2569-10-28_at_12.40.24_PM.png)
    

#### Task 2

1. **Email address analysis.** The structure of an email address (username, `@` symbol, domain) was studied, and the recipient domain of the example address `hatsalesman@tryhatme.com` was identified.
    
    so the answer was `tryhatme.com` 
    

#### Task 3

There are 3 protocols involved in the sending and receiving of email messages. They are listed below.

- SMTP (Simple Mail Transfer Protocol) – Used to send emails.
- POP3 (Post Office Protocol) – Used to transfer emails between a client and server.
- IMAP (Internet Message Access Protocol) – Used to transfer emails between a client and server.

As you can see POP3 and IMAP have the same definition but there are some differences between them.

**POP3:**

- Emails are downloaded and stored to one device.
- Sent Emails are stored to one device.
- Emails can only be accessed from one device.
- To keep messages on the server make sure that ‘keep email on server’ is turned on, otherwise all messages will be deleted from the server when they have been downloaded to your one device.

**IMAP:**

- Emails are stored on the server and can be downloaded to multiple devices.
- Sent Messages are stored on the server.
- Messages can be synced across multiple devices.

**Questions** 

1. Which protocol is responsible for sending an email from a client to a mail server?
    
    `SMPT` 
    
2. Which service is used to look up the recipient domain’s mail server?
    
    `DNS`
    
3. Bob wants to access his email from multiple devices, including his phone and laptop. Which protocol should he use?
    
    `IMAP`
    

#### Task 4

An email consists of two main parts:

- **Email header**: Contains metadata about the message, such as sender and the servers involved in delivery
- **Email body**: Contains the actual message content, which may be plain text or HTML

Let's take a look at the different components that make up an email header.

1. **From**: The sender's email address
2. **To**: The receiver's email address
3. **Reply to**: Address where replies are sent (not required)
4. **Subject**: The email's subject line
5. **Date**: The time and date that the email was sent

**Questions**

1. What is the full subject line of `email1.eml`?
    
    Help protect your budget by protecting your home
    
2. View the message source of `email1.eml`What the IP address listed as the `X-Originating-Ip` ? 

![Screenshot 2569-10-28 at 6.11.28 PM.png](Lab%205/Screenshot_2569-10-28_at_6.11.28_PM.png)

#### Task 5

**Email Body** 

The body of an email contains the message. Emails are sent as text only or formatted in HTML. HTML supports elements such as images, links, and styling. Most email clients show you the rendered content. You can also inspect the underlying source to see how the message is structured, spot embedded elements, and look for signs of phishing or malicious content.

**Viewing HTML Source Code**

Just as we can inspect an email’s source to analyze its headers, we can also examine the source of the email body. 

![image.png](Lab%205/image.png)

**Reconstructing Attachments**

mails can also include attachments, such as documents, images, or other file types. Just as the email body can be analyzed by viewing the message source, attachments can be analyzed as well. This allows us to understand how the file is embedded within the email.

When we inspect the source, we can identify several important headers associated with the attachment:

- **Content-Type** indicates the file type `application/pdf`
- **Content-Disposition** specifies that the file is an attachment and includes its filename
- **Content-Transfer-Encoding** shows that the file is `base64` encoded

![image.png](Lab%205/image%201.png)

**Question** 

1. Open up the `email2.txt` file to view the source of an attachment. What is the `Content-Type` of the attachment?
    
    application/pdf
    
2. What is the name of the attachment from the previous question?
    
    `zmqpalgh.pdf`
    
3.  Decode the base64 string using either a [PDF converter](https://www.apivoid.com/tools/base64-to-pdf/) or [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)) . What is the hidden flag value?
    
    THM{BENIGN_PDF_ATTACHM
    

ENT}

![Screenshot 2569-10-28 at 6.23.50 PM.png](Lab%205/Screenshot_2569-10-28_at_6.23.50_PM.png)

#### Task 6

**Type of Phishing** 

- [Spam](https://www.proofpoint.com/us/threat-reference/spam): Unsolicited bulk emails sent to a large number of recipients. A more malicious form of spam is often called malspam.
- [Phishing](https://www.proofpoint.com/us/threat-reference/phishing): Emails that impersonate a trusted entity to trick recipients into revealing sensitive information.
- [Spear Phishing](https://www.proofpoint.com/us/threat-reference/spear-phishing): A targeted form of phishing aimed at a specific individual or organization, often using personalized information.
- [Whaling](https://www.rapid7.com/fundamentals/whaling-phishing-attacks/): A type of spear phishing that specifically targets high-level executives (CEO, CFO) to obtain sensitive data or financial access.
- [Smishing](https://www.proofpoint.com/us/threat-reference/smishing): Phishing attacks conducted via SMS or text messages, targeting users on mobile devices.
- [Vishing](https://www.proofpoint.com/us/threat-reference/vishing): Phishing attacks carried out through voice calls, where attackers use social engineering over the phone.

**Anatomy of a Phishing Email**

- **Spoofed From Address**: The sender’s email is spoofed to appear as a trusted entity (`noreply@microsof.com`)
- **Urgent Subject or Message**: The email creates a sense of urgency (“Your account will be locked in 24 hours”)
- **Brand Impersonation**: The email is designed to mimic a legitimate organization (logos or colors matching a real company)
- **Grammar & Spelling Issues**: The message may contain errors, though with AI these are now less common (awkward phrasing or unnatural wording)
- **Generic Content**: The message lacks personalization (“Dear Customer” instead of your name)
- **Hidden or Shortened Links**: Hyperlinks may disguise their true destination (`bit.ly/secure-login`)
- **Malicious Attachments**: Attachments are included and disguised as legitimate files (`invoice.pdf.exe`)

**Safe Analysis**

When dealing with hyperlinks and attachments, be careful not to click them accidentally. [Hyperlinks](https://gchq.github.io/CyberChef/#recipe=Defang_URL(true,true,true,'Valid%20domains%20and%20full%20URLs')) and [IP addresses](https://gchq.github.io/CyberChef/#recipe=Defang_IP_Addresses()) should be **defanged**. Defanging makes URLs, domains, or email addresses unclickable to prevent accidental clicks that could lead to a security breach. It works by replacing special characters, such as `@` in an email or `.` in a URL, with alternate characters:

- Original URL: `http://www.suspiciousdomain.com`
- Defanged URL: `hxxp[://]www[.]suspiciousdomain[.]com`

**Questions**

1. Which reputable organization is being spoofed in this phishing attempt?
    
    Home Depot
    
2. What is the sender's email address?
    
    [support@teckbe.com](mailto:support@teckbe.com)
    
3. Inspect the email message source.What is the [defanged](https://gchq.github.io/CyberChef/#recipe=Defang_IP_Addresses()) `X-Originating-IP`?
    
    `103[.]234[.]236[.]83` 
    
4. Continue analyzing the email message source. Which mail server generated the `Authentication-Results` header?
    
    [atlas102.free.mail.gq1.yahoo.com](http://atlas102.free.mail.gq1.yahoo.com/)
    

#### Task 7

Business Email Compromise (BEC) is a form of cybercrime in which attackers use a
compromised email account, impersonation, or a similar deception to persuade employees
or other victims to perform unauthorized actions.

These actions may include transferring money, disclosing confidential information, or
changing payment details.

**Question** 

1. What attack, signified by the acronym BEC, uses a compromised email to trick
employees into fraud?
    
    Business Email Compromise
    

### **5.2 Lab 2: Windows Forensics 1**

#### Task 1

**Introduction to Computer Forensics for Windows**

Computer forensics is an essential field of cyber security that involves gathering evidence of activities performed on computers. It is a part of the wider Digital Forensics field, which deals with forensic analysis of all types of digital devices, including recovering, examining, and analyzing data found in digital devices.

**Question**

1. What is the most used Desktop Operating System right now?
    
    Microsoft Windows
    

#### Task 2

**Windows Registry**

The Windows Registry is a collection of databases that contains the system's configuration data. This configuration data can be about the hardware, the software, or the user's information. It also includes data about the recently used files, programs used, or devices connected to the system. 

The registry on any Windows system contains the following five root keys:

1. `HKEY_CURRENT_USER`
2. `HKEY_USERS`
3. `HKEY_LOCAL_MACHINE`
4. `HKEY_CLASSES_ROOT`
5. `HKEY_CURRENT_CONFIG`

| **Folder/predefined key** | **Description** |
| --- | --- |
| **HKEY_CURRENT_USER** | Contains the root of the configuration information for the user who is currently logged on. The user's folders, screen colors, and Control Panel settings are stored here. This information is associated with the user's profile. This key is sometimes abbreviated as HKCU. |
| **HKEY_USERS** | Contains all the actively loaded user profiles on the computer. HKEY_CURRENT_USER is a subkey of HKEY_USERS. HKEY_USERS is sometimes abbreviated as HKU. |
| **HKEY_LOCAL_MACHINE** | Contains configuration information particular to the computer (for any user). This key is sometimes abbreviated as HKLM. |
| **HKEY_CLASSES_ROOT** | Is a subkey of `HKEY_LOCAL_MACHINE\Software` . The information that is stored here makes sure that the correct program opens when you open a file by using Windows Explorer. This key is sometimes abbreviated as HKCR.
Starting with Windows 2000, this information is stored under both the HKEY_LOCAL_MACHINE and HKEY_CURRENT_USER keys. The `HKEY_LOCAL_MACHINE\Software\Classes` key contains default settings that can apply to all users on the local computer.  The `HKEY_CURRENT_USER\Software\Classes` key has settings that override the default settings and apply only to the interactive user.
The HKEY_CLASSES_ROOT key provides a view of the registry that merges the information from these two sources. HKEY_CLASSES_ROOT also provides this merged view for programs that are designed for earlier versions of Windows. To change the settings for the interactive user, changes must be made under `HKEY_CURRENT_USER\Software\Classes` instead of under HKEY_CLASSES_ROOT.
To change the default settings, changes must be made under `HKEY_LOCAL_MACHINE\Software\Classes`  .If you write keys to a key under HKEY_CLASSES_ROOT, the system stores the information under `HKEY_LOCAL_MACHINE\Software\Classes` .
If you write values to a key under HKEY_CLASSES_ROOT, and the key already exists under `HKEY_CURRENT_USER\Software\Classes` , the system will store the information there instead of under `HKEY_LOCAL_MACHINE\Software\Classes` . |
| **HKEY_CURRENT_CONFIG** | Contains information about the hardware profile that is used by the local computer at system startup. |

**Question** 

1. What is the short form for HKEY_LOCAL_MACHINE?
    
    `HKLM`
    

#### Task 3

Accessing a live system, we will be able to access the registry using regedit.exe, and we will be greeted with all of the standard root keys we learned about in the previous task. However, if we only have access to a disk image, we must know where the registry hives are located on the disk. The majority of these hives are located in the `C:\Windows\System32\Config` directory and are:

1. **DEFAULT** (mounted on `HKEY_USERS\DEFAULT`)
2. **SAM** (mounted on `HKEY_LOCAL_MACHINE\SAM`)
3. **SECURITY** (mounted on `HKEY_LOCAL_MACHINE\Security`)
4. **SOFTWARE** (mounted on `HKEY_LOCAL_MACHINE\Software`)
5. **SYSTEM** (mounted on `HKEY_LOCAL_MACHINE\System`)

**Hives containing user information:**

Apart from these hives, two other hives containing user information can be found in the User profile directory. For Windows 7 and above, a user’s profile directory is located in `C:\Users\<username>\` where the hives are:

1. **NTUSER.DAT** (mounted on HKEY_CURRENT_USER when a user logs in)
2. **USRCLASS.DAT** (mounted on HKEY_CURRENT_USER\Software\CLASSES)

The USRCLASS.DAT hive is located in the directory `C:\Users<username>\AppData\Local\Microsoft\Windows`.

The NTUSER.DAT hive is located in the directory `C:\Users\<username>\`.

**Amcache Hive:**

Apart from these files, there is another very important hive called the AmCache hive. This hive is located in `C:\Windows\AppCompat\Programs\Amcache.hve`. Windows creates this hive to save information on programs that were recently run on the system.

Questions 

1. What is the path for the five main registry hives, DEFAULT, SAM, SECURITY, SOFTWARE, and SYSTEM?
    
    `C:\Windows\System32\Config` 
    
2. What is the path for the AmCache hive?
    
    `C:\Windows\AppCompat\Programs\Amcache.hve`
    

#### Task 4

**Data Acquisition** 

When performing forensics, we will either encounter a live system or an image taken of the system. For the sake of accuracy, it is recommended practice to image the system or make a copy of the required data and perform forensics on it. This process is called data acquisition.

we can use one of the following tools:

- [KAPE](https://www.kroll.com/en/services/cyber-risk/incident-response-litigation-support/kroll-artifact-parser-extractor-kape) is a live data acquisition and analysis tool which can be used to acquire registry data.
- [Autopsy](https://www.autopsy.com/)  gives you the option to acquire data from both live systems or from a disk image.
- [FTK Imager](https://www.exterro.com/ftk-imager)  is similar to Autopsy and allows you to extract files from a disk image or a live system by mounting the said disk image or drive in FTK Imager.

#### Task 5

**Exploring Windows Registry**

Once we have extracted the registry hives, we need a tool to view these files as we would in the registry editor. Since the registry editor only works with live systems and can't load exported hives, we can use the following tools:

- **Registry Viewer:**  [Registry Viewer](https://accessdata.com/product-download/registry-viewer-2-0-0)  only loads one hive at a time, and it can't take the transaction logs into account.
- **Zimmerman's Registry Explorer:** Eric Zimmerman has developed a handful of [tools](https://ericzimmerman.github.io/#!index.md) that are very useful for performing Digital Forensics and Incident Response.
- **RegRipper:**  [RegRipper](https://github.com/keydet89/RegRipper3.0) is a utility that takes a registry hive as input and outputs a report that extracts data from some of the forensically important keys and values in that hive. The output report is in a text file and shows all the results in sequential order.

#### Task 6

**System Information and System Accounts**

**OS Version:**

The following were read from the registry:

- OS build (`SOFTWARE\Microsoft\Windows NT\CurrentVersion`)
- current and last-known-good control sets (`SYSTEM\Select`),
- computer name (`SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName`),
- time zone (`...\Control\TimeZoneInformation`),
- network interfaces (`...\Services\Tcpip\Parameters\Interfaces`),
- past networks (`...\NetworkList\Signatures`) and autostart entries (`Run` and `RunOnce` keys).

**Questions**

1. What is the Current Build Number of the machine whose data is being  investigated?
Ans: 19044
2. Which ControlSet contains the last known good configuration?
Ans: 1
3. What is the Computer Name of the computer?
Ans: THM-4n6
4. What is the value of the TimeZoneKeyName?
Ans: Pakistan Standard Time
5. What is the DHCP IP address
Ans: 192.168.100.58
6. What is the RID of the Guest User account?
Ans: 501

#### Task 7

**Usage or knowledge of files/folders**

1. **Recent Files:**
    
    Windows maintains a list of recently opened files for each user. This information is stored in the `NTUSER` hive and can be found on the following location:
    
    `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`
    
    Another interesting piece of information in this registry key is that there are different keys with file extensions, such as  `.pdf` , `.jpg` , `.docx`  etc. These keys provide us with information about the last used files of a specific file extension. So if we are looking specifically for the last used PDF files, we can look at the following registry key:
    
    `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\.pdf`
    
2. **Office Recent Files:**
    
    Similar to the Recent Docs maintained by Windows Explorer, Microsoft Office also maintains a list of recently opened documents. It can be found in the following location:
    
    `NTUSER.DAT\Software\Microsoft\Office\VERSION`
    
    The version number for each Microsoft Office release is different. An example registry key will look like this:
    
    `NTUSER.DAT\Software\Microsoft\Office\15.0\Word`
    
3. **ShellBags:**
    
    Information about the Windows *'shell'*  is stored and can identify the Most Recently Used files and folders. Since this setting is different for each user, it is located in the user hives. We can find this information on the following locations:
    
    `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags`
    
    `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU`
    
    `NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU`
    
    `NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags`
    
4. **Open/Save and LastVisited Dialog MRUs:**
    
    When we open or save a file, a dialog box appears asking us where to save or open that file from. It might be noticed that once we open/save a file at a specific location, Windows remembers that location. This implies that we can find out recently used files if we get our hands on this information. We can do so by examining the following registry keys
    
    `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePIDlMRU`
    
    `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU`
    

1. **Windows Explorer Address/Search Bars:**
    
    Another way to identify a user's recent activity is by looking at the paths typed in the Windows Explorer address bar or searches performed using the following registry keys, respectively.
    
    `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths`
    
    `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery`
    

**Questions**

1. When were EZtools opened?
Ans: 2021-12-01 13:00:34
2. When were EZtools opened?
Ans: 2021-12-01 13:06:47
3. What is the Absolute Path of the file opened using notepad.exe?
Ans: C:\Program Files\Amazon\Ec2ConfigService\Settings
4. When was this file opened?
Ans: 2021-11-30 10:56:19

#### Task 8

**Evidence of Execution**

1. **UserAssist:** 
    
    Windows keeps track of applications launched by the user using Windows Explorer for statistical purposes in the User Assist registry keys. These keys contain information about the programs launched, the time of their launch, and the number of times they were executed. However, programs that were run using the command line can't be found in the User Assist keys. The User Assist key is present in the NTUSER hive, mapped to each user's GUID. We can find it at the following location:
    
    `NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\{GUID}\Count`
    
2. **ShimCache:**
    
    ShimCache is a mechanism used to keep track of application compatibility with the OS and tracks all applications launched on the machine. Its main purpose in Windows is to ensure backward compatibility of applications. It is also called Application Compatibility Cache (AppCompatCache). It is located in the following location in the SYSTEM hive:
    
    `SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache`
    
    ShimCache stores file name, file size, and last modified time of the executables.
    
3. **AmCache:**
    
    The AmCache hive is an artifact related to ShimCache. This performs a similar function to ShimCache, and stores additional data related to program executions. This data includes execution path, installation, execution and deletion times, and SHA1 hashes of the executed programs. This hive is located in the file system at:
    
    `C:\Windows\appcompat\Programs\Amcache.hve`
    
    Information about the last executed programs can be found at the following location in the hive:
    
    `Amcache.hve\Root\File\{Volume GUID}\`
    
4. **BAM/DAM:**
    
    Background Activity Monitor or BAM keeps a tab on the activity of background applications. Similar Desktop Activity Moderator or DAM is a part of Microsoft Windows that optimizes the power consumption of the device. Both of these are a part of the Modern Standby system in Microsoft Windows.
    
    In the Windows registry, the following locations contain information related to BAM and DAM. This location contains information about last run programs, their full paths, and last execution time.
    
    `SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}`
    
    `SYSTEM\CurrentControlSet\Services\dam\UserSettings\{SID}`
    

**Questions**

1. How many times was the File Explorer launched?
Ans: 26
2. What is another name for ShimCache?
Ans: AppCompatCache
3. Which of the artifacts also saves SHA1 hashes of the executed programs?
Ans: AmCache
4. Which of the artifacts saves the full path of the executed programs?
Ans: BAM/DAM

#### Task 9

1. **Device identification:**
    
    The following locations keep track of USB keys plugged into a system. These locations store the vendor id, product id, and version of the USB device plugged in and can be used to identify unique devices. These locations also store the time the devices were plugged into the system.
    
    `SYSTEM\CurrentControlSet\Enum\USBSTOR`
    
    `SYSTEM\CurrentControlSet\Enum\USB`
    
2. **First/Last Times:**
    
    the following registry key tracks the first time the device was connected, the last time it was connected and the last time the device was removed from the system.
    
    `SYSTEM\CurrentControlSet\Enum\USBSTOR\Ven_Prod_Version\USBSerial#\Properties\{83da6326-97a6-4088-9453-a19231573b29}\####` 
    
    In this key, the #### sign can be replaced by the following digits to get the required information:
    
    | **Value** | **Information** |
    | --- | --- |
    | 0064 | First Connection time |
    | 0066 | Last Connection time |
    | 0067 | Last removal time |
3. **USB device Volume Name:**
    
    The device name of the connected drive can be found at the following location:
    
    `SOFTWARE\Microsoft\Windows Portable Devices\Devices` 
    

**Questions**

1. What is the serial number of the device from the manufacturer 'Kingston'?
Ans: 1C6f654E59A3B0C179D366AE&0
2. What is the name of this device?
Ans: Kingston Data Traveler 2.0 USB Device
3. What is the friendly name of the device from the manufacturer 'Kingston'?
Ans: USB

#### Task 10

**Scenario:**

One of the Desktops in the research lab at Organization X is suspected to have been accessed by someone unauthorized. Although they generally have only one user account per Desktop, there were multiple user accounts observed on this system. It is also suspected that the system was connected to some network drive, and a USB device was connected to the system. The triage data from the system was collected and placed on the attached VM. Can you help Organization X with finding answers to the below questions?

**Note:** When loading registry hives in RegistryExplorer, it will caution us that the hives are dirty. This is nothing to be afraid of. We just need to remember the little lesson about transaction logs and point RegistryExplorer to the .LOG1 and .LOG2 files with the same filename as the registry hive. It will automatically integrate the transaction logs and create a 'clean' hive. Once we tell RegistryExplorer where to save the clean hive, we can use that for our analysis and we won't need to load the dirty hives anymore. RegistryExplorer will guide you through this process.

**Questions**

1. How many user created accounts are present on the system?
    
    Ans: 3
    
2. What is the username of the account that has never been logged in?
Ans: thm-user2
3. What's the password hint for the user THM-4n6?
Ans: count
4. When was the file 'Changelog.txt' accessed?
Ans: 2021-11-24 18:18:48
5. What is the complete path from where the python 3.8.2 installer was run?
Ans: Z:\setups\python-3.8.2.exe
6. When was the USB device with the friendly name 'USB' last connected?
Ans: 2021-11-24 18:40:06

## 6. Conclusion

Both laboratories met their stated objectives. Together they show the two halves of an incident response workflow: recognising a malicious message at the point of entry, and reconstructing what a user or attacker did on a host afterwards.

**Phishing analysis.** The first lab showed that the raw message source holds evidence that the inbox view hides, including the originating IP address, authentication results and encoded attachments. It also showed that no single indicator is conclusive. A spoofed sender, an urgent tone, a generic greeting and a disguised link are most reliable when they are considered together. Defanging indicators before recording them is a simple safeguard against accidental compromise during analysis.

**Registry forensics.** The second lab showed that Windows records a detailed trail of user activity as a by-product of personalising the user experience. System identity, accounts, recent files, executed programs and USB devices can each be traced to specific registry locations. Working from a triage copy, and merging transaction logs before analysis, preserves the original evidence and gives more current results. Correlating several artifacts, and fixing the time zone first, is what allows a reliable timeline to be built.

**Limitations.** Both labs used a small, curated set of samples in a controlled environment. Real investigations involve larger volumes of data, anti-forensic activity, and the need to document chain of custody. Artifacts such as ShimCache and UserAssist show that a program was present or launched, but not always who ran it or why, so findings should be corroborated from more than one source. The Windows lab also depends on registry artifacts alone and does not cover other sources such as event logs or memory.

**Recommendations.** For defence, organisations should combine user awareness training with email authentication controls and a clear reporting path for suspicious messages. For investigation, analysts should acquire evidence forensically, record time zones, and cross-check findings across artifacts. Further study could follow the room series, including Phishing Emails in Action, Phishing Analysis Tools, Windows Forensics 2 and the KAPE room.