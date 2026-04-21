# 01 - Lab Setup: Building Your Windows 11 Practice Environment

---

## 1. What This File Is About

Before you can practice a single privilege escalation technique, you need a safe place to practice it. That place is a virtual machine - a Windows 11 computer running inside your real computer, completely isolated from the outside world.

You already have Kali Linux running on VMware Workstation Pro. This file will walk you through adding a Windows 11 virtual machine right alongside it, configuring the two machines to talk to each other, and loading every tool used in this course onto the Windows machine.

By the end of this file you will have:

- A fully installed Windows 11 Enterprise virtual machine
- VMware Tools installed (makes the VM usable and shareable)
- A network setup where your Kali machine can reach your Windows machine
- Windows Defender completely turned off
- Automatic Windows updates disabled
- A low-privilege user account called `labuser` for practicing privilege escalation
- Every tool used in this course installed and ready to go
- Three saved snapshots so you can reset the machine instantly whenever something breaks

This is the only setup file in the course. Once this is done, every future file will tell you exactly which tools to use on which machine. You will never need to set up the lab again.

---

## 2. Download Windows 11 Enterprise Evaluation ISO

Microsoft gives away a free 90-day trial version of Windows 11 Enterprise specifically so that IT professionals, security researchers, and students can test things without buying a license. This is completely legitimate. You do not need a license key. The machine works fully for 90 days and can be extended further for lab purposes.

### Step 1 - Go to the download page

Open a browser on your main machine (not inside any VM) and go to this exact address:

```
https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise
```

You will land on a page titled **Windows 11 Enterprise** with a blue or dark banner and a button that says something like **Start your evaluation** or **Get started for free**.

### Step 2 - Register and start the download

Click the **Start your evaluation** button. Microsoft will ask you to fill in a short registration form before giving you the download. Fill in the fields like this:

| Field | What to put |
|---|---|
| First name | Your first name |
| Last name | Your last name |
| Work email | Any valid email address you check |
| Country/Region | India |
| Organization size | 1-24 employees (pick any option) |
| Organization | You can put anything here - "Lab" works |
| Job title | Student / Security Researcher |
| Phone number | Your phone number |

You do not have to opt into their marketing emails. Uncheck those boxes if they appear.

Click **Continue** or **Next** after filling the form.

### Step 3 - Choose the download type

On the next page you will see options for what you want to download. You want:

- **ISO - Enterprise** (not VHD or Azure)
- **Language:** English (United States)
- **Architecture:** 64-bit (x64)

Click the **Download** button.

### Step 4 - Wait for the download to finish

The file is called something like `Windows11_InsiderPreview_EnterpriseVHDX_x64_en-us.iso` or `Win11_Eval_x64FRE_en-us.iso`. It is approximately **5.5 to 6 GB** in size.

On a typical Indian broadband connection at 50–100 Mbps, this will take between **10 and 30 minutes**. On slower connections it may take longer. Save it somewhere you will remember - your Downloads folder or a folder you create called `C:\ISOs\` is fine.

### What the 90-day evaluation means

During the 90 days, Windows 11 works completely normally - all features, all functionality. After 90 days the machine will start showing a watermark in the corner saying the evaluation has expired, and it will begin shutting itself down every hour. For a lab machine that you will reset with snapshots regularly, this is not a real problem because you will restore snapshots long before 90 days pass. You can also extend the evaluation once using the command `slmgr /rearm` in an elevated command prompt, which adds another 90 days.

---

## 3. Create the Windows 11 VM in VMware Workstation Pro

Now you will tell VMware to create a new virtual machine and point it at the ISO you just downloaded. This is like building a new computer - you decide how much RAM, how many CPU cores, and how much hard drive space to give it.

### Step 1 - Open VMware Workstation Pro

Launch VMware Workstation Pro on your main machine. You will see the main VMware window with your existing Kali Linux VM listed on the left side.

### Step 2 - Start the new VM wizard

Click **File** in the top menu bar, then click **New Virtual Machine**. A dialog box called the New Virtual Machine Wizard will open.

When asked which configuration type you want, select **Custom (advanced)** and click **Next**. This gives you more control over the settings, which you need for the TPM setup.

### Step 3 - Hardware compatibility

The next screen asks you about hardware compatibility. Leave this at the default - it will say something like **Workstation 17.x** or the latest version you have installed. Click **Next**.

### Step 4 - Select the installer ISO

On the screen that asks **How will you install the guest operating system**, choose **Installer disc image file (iso)** and click **Browse**. Navigate to where you saved the Windows 11 Enterprise ISO and select it.

VMware may or may not detect it automatically. If it shows **Windows 11** as the detected OS, great. If it shows **Other** or something unexpected, that is fine - we will set it manually in the next step. Click **Next**.

### Step 5 - Select the guest operating system

On this screen, make sure these are selected:

- **Guest operating system:** Microsoft Windows
- **Version:** Windows 11 x64

If VMware does not show Windows 11 as an option, select **Windows 10 x64** - it will still work. Click **Next**.

### Step 6 - Name your VM and choose where to save it

Give the VM a name. Call it something clear like:

```
Windows 11 - Lab
```

For the location, VMware will suggest a default folder. You can keep the default or change it to a drive that has plenty of space - you need at least 100 GB free on whatever drive you pick. Click **Next**.

### Step 7 - Firmware type - UEFI is required

This is important. On the screen that asks about **Firmware Type**, you must select **UEFI**. Windows 11 requires UEFI. Do not select BIOS.

You will also see a checkbox or option for **Enable Secure Boot**. Leave this **checked** for now. Click **Next**.

### Step 8 - Assign CPU cores

On the processor configuration screen, set:

- **Number of processors:** 1
- **Number of cores per processor:** 4

This gives the VM 4 total CPU cores, which is more than enough. Click **Next**.

### Step 9 - Assign RAM

On the memory screen, set the VM memory to **8192 MB** (which is 8 GB).

You have 24 GB total RAM on your main machine. Kali Linux uses about 2–4 GB while running. Windows 11 will use 4–6 GB. Your host machine needs about 4 GB to stay stable. 8 GB for Windows is a good balance. Click **Next**.

### Step 10 - Network type

On the **Network Type** screen, select **Use network address translation (NAT)**. We will change this later when setting up the lab network, but NAT is the safest choice during installation because it gives the VM internet access (needed to download things) while keeping it somewhat isolated. Click **Next**.

### Step 11 - I/O Controller and disk type

For the **I/O Controller Type** screen, leave the recommended option selected (it will say something like **LSI Logic SAS** or **PVSCSI**). Click **Next**.

For the **Virtual Disk Type** screen, leave the recommended option selected (**NVMe** or **SCSI**). Click **Next**.

### Step 12 - Create the virtual hard disk

On the disk screen, select **Create a new virtual disk**. Click **Next**.

On the disk size screen, set the disk size to **80 GB**. Also select **Split virtual disk into multiple files** - this is easier to manage than a single large file. Click **Next**.

On the next screen, VMware will suggest a filename for the disk file. Leave it as is. Click **Next**.

### Step 13 - Review settings before you finish

You will now see a summary of all your VM settings. Before clicking Finish, click the **Customize Hardware** button. This opens the hardware editor.

Here you need to remove the **Floppy** drive if it appears - right-click it and click **Remove**. Also remove the **Sound Card** if you see it, since you do not need it for a lab machine.

Do not close this hardware editor yet - you have one more important thing to do here.

### Step 14 - Add the TPM device (the VMware method)

Windows 11 requires a TPM 2.0 chip. Your VM does not have one by default. VMware Workstation Pro has a built-in way to add a software-based virtual TPM, but it requires the VM to be encrypted first.

Here is how to do it, right here in the hardware editor:

**14a.** In the hardware editor, click the **Options** tab (at the top, next to "Hardware").

**14b.** In the left column under Options, click on **Access Control**.

**14c.** On the right side, click the **Encrypt** button.

**14d.** VMware will ask you to set an encryption password. This password is NOT the Windows login password - it is just VMware's internal encryption key for the VM files. Choose something simple that you will not forget, like `LabEncrypt123`. Type it in both boxes and click **Encrypt**.

The encryption happens quickly. You will see a progress bar and then it will return to the Options screen.

**14e.** Now click back on the **Hardware** tab.

**14f.** Click the **Add** button at the bottom left of the hardware list.

**14g.** In the Add Hardware Wizard that opens, look for **Trusted Platform Module** in the list and click on it. Then click **Finish**.

You will now see **Trusted Platform Module** listed in the VM hardware. This is your virtual TPM 2.0.

**14h.** Click **Close** to close the hardware editor, then click **Finish** on the wizard.

Your Windows 11 VM is now created and has a virtual TPM. VMware will show it in your VM list on the left side.

---

## 4. Install Windows 11 Step by Step

Now you will actually install Windows 11 inside the VM. This part is like setting up a brand new computer.

### Step 1 - Power on the VM

Click on your **Windows 11 - Lab** VM in the VMware library on the left, then click the green **Power on this virtual machine** button.

The VM will start and show a black screen for a few seconds, then you will see a message at the bottom that says **Press any key to boot from CD or DVD...**

Click inside the VM window to capture your mouse and keyboard, then **press any key immediately** (like the spacebar). If you miss this prompt, the VM will try to boot from disk and show an error. If that happens, power off the VM and try again.

### Step 2 - Language and keyboard selection

After a short loading screen with the Windows logo, you will see the Windows Setup screen. It shows three dropdowns:

- **Language to install:** English (United States)
- **Time and currency format:** English (India) - you can change this here or set it later in Windows. Either is fine.
- **Keyboard or input method:** US

Leave these at defaults or adjust as you prefer. Click **Next**, then click **Install now**.

### Step 3 - Activate Windows screen

Windows will ask you for a product key. Click the small link at the bottom that says **I don't have a product key**. This is correct - the evaluation ISO does not need a key. Windows will activate itself automatically based on the evaluation license.

### Step 4 - Select the Windows edition

A list of editions will appear:

- Windows 11 Home
- Windows 11 Home N
- Windows 11 Pro
- Windows 11 Pro N
- **Windows 11 Enterprise** ← select this one
- Windows 11 Enterprise N
- Windows 11 Education

Select **Windows 11 Enterprise** and click **Next**.

### Step 5 - Accept the license terms

Read or skim the license agreement. Check the box that says **I accept the Microsoft Software License Terms** and click **Next**.

### Step 6 - Installation type

When asked which type of installation you want, click **Custom: Install Windows only (advanced)**. You are doing a fresh install on a brand new virtual disk, not upgrading anything.

### Step 7 - Disk partitioning

You will see a list of available disk space. It will show one entry: **Drive 0 Unallocated Space** with approximately 80 GB. Click on it to select it, then click **Next**.

Windows will create partitions automatically and begin installing. You will see a progress screen showing several stages:

```
Copying Windows files      ✓
Preparing files for installation  ✓
Installing features        ...
Installing updates         ...
Finalizing installation    ...
```

This takes between **10 and 25 minutes** depending on your machine speed. The VM will restart once or twice during this process. Let it restart - do not press any key when it shows the "Press any key to boot from CD..." prompt again. Just let it continue on its own.

### Step 8 - The setup experience (OOBE) - creating a local account

After the restarts, Windows will walk you through the initial setup. This is called OOBE (Out of Box Experience). The screens look like a friendly setup wizard with a dark background and large text.

The first few screens will ask you about:

**Country/Region:** Select **India** (or your actual country). Click **Yes**.

**Keyboard layout:** Select **US**. Click **Yes**, then click **Skip** on the second keyboard layout screen.

**Network screen:** Windows will ask you to connect to a network or Wi-Fi. You will see a list of networks or it may auto-detect VMware's NAT connection. If the internet connects automatically, fine. If not, click **I don't have internet** at the bottom left - this will appear as a small link.

**This is important - the Microsoft account step:**

Windows 11 will now push hard for you to sign in with a Microsoft account. The screen will say something like **Let's add your Microsoft account** or **Sign in with Microsoft**. You do not want this - you want a plain local account.

Here is the method that works on Windows 11 Enterprise in 2024 and 2025:

On the Microsoft account screen, look for a link at the bottom left that says **Sign-in options**. Click it.

Then look for a link that says **Domain join instead**. Click it.

This will ask you to create a local account without needing a Microsoft account. Type the username:

```
labadmin
```

Then set a password:

```
Admin@Lab123
```

Confirm the password. Then Windows will ask you to set three security questions and answers. Pick any three and write them down or type simple answers you will remember.

> **Why `labadmin` and not just `admin`?** Windows already has a built-in `Administrator` account. Using `labadmin` as your admin account name avoids any confusion between the two.

Click **Next** after each screen.

### Step 9 - Privacy settings

Windows will show a screen with a list of privacy options, each with a toggle:

- Location: **Off**
- Find my device: **Off**
- Diagnostic data: **Required only** (the minimum option)
- Inking & typing improvement: **Off**
- Tailored experiences: **Off**
- Advertising ID: **Off**

Turn everything to the minimum or off. This is a lab machine - you do not need any of this. Click **Accept**.

### Step 10 - Let Windows finish

Windows will now spend a few more minutes finishing setup. You will see screens saying things like **Hi`, and then a loading screen. Wait for it to complete. Eventually the Windows 11 desktop will appear.

You have successfully installed Windows 11. The desktop will look like a clean Windows install with a taskbar, Start button, and a wallpaper.

---

## 5. Install VMware Tools

Right now the VM works but it has a few annoying limitations: the screen resolution is probably very low (like 800x600), you cannot copy and paste text between your main machine and the VM, and your mouse may not move smoothly in and out of the VM. VMware Tools fixes all of this.

### What VMware Tools actually is

VMware Tools is a small software package that VMware installs inside the guest VM. It acts as a communication bridge between the host machine and the VM. Once installed, the VM knows it is running inside VMware and takes advantage of that - the graphics driver switches to a proper display adapter, copy-paste starts working, and folder sharing becomes possible. Every VM you ever create in VMware should have VMware Tools installed.

### Step 1 - Mount the VMware Tools installer

At the top of the VMware window (while the Windows 11 VM is running), click **VM** in the menu bar, then hover over **Install VMware Tools...** and click it.

This mounts a virtual CD inside the Windows 11 VM, as if you inserted a disc. The VM may show an AutoPlay popup asking what to do with the new disc. If it does, click **Run setup64.exe**.

If AutoPlay does not appear, go inside the Windows 11 VM, open **File Explorer** (the folder icon in the taskbar), look for a **DVD Drive** listed on the left side (it will say something like **DVD Drive (D:) VMware Tools**), and double-click it. Then double-click the file called **setup64.exe** inside it.

### Step 2 - Run the installer

The VMware Tools installer will open. You will see a welcome screen. Click **Next**.

On the setup type screen, select **Typical** and click **Next**.

Click **Install**. The installation will take about 1–2 minutes.

When it finishes, click **Finish**. Windows will ask you to **Restart Now** to complete the installation. Click **Yes**.

### Step 3 - After the restart

When Windows restarts and you log back in with `labadmin`, you will notice:

- The screen resolution has jumped to something reasonable (like 1280x800 or higher)
- You can now resize the VMware window and the resolution will adjust automatically
- Moving your mouse in and out of the VM is seamless - no need to press Ctrl+Alt to release the mouse
- Copy and paste between your main machine and the VM now works

VMware Tools is installed. Moving on.

---

## 6. Configure the Network for Your Lab

Right now the Windows 11 VM is set to NAT networking. That means the VM can access the internet through your host machine. That is useful for downloading tools but it is not ideal for red team lab work.

For running attacks, you want the Kali VM and the Windows 11 VM to be on the same isolated private network - a network where they can talk to each other directly, but neither VM is directly reachable from the internet or your home network. This is called **Host-Only** networking.

### What NAT means in plain words

NAT (Network Address Translation) is like your home router. When the VM wants to reach the internet, the VM's traffic goes through VMware, which translates the VM's private IP address to your host machine's real IP address, and sends the traffic out. The internet sees the host machine's IP, not the VM's IP. The VM can reach the outside world, but nothing from the outside can reach the VM directly.

This is good for downloading software. It is not ideal for running attack tools between two VMs, because the two VMs are on slightly different network segments.

### What Host-Only means in plain words

Host-Only creates a completely private network that exists only inside VMware. No traffic from this network goes to the internet or your home router. Any VMs on the same Host-Only network can talk to each other, and they can also talk to the host machine - but they cannot reach anything outside your physical computer.

This is exactly what you want for a red team lab. You can throw attacks, run exploitation tools, make as much noise as you want, and none of it leaves your machine.

### The plan for this lab

You will set both VMs - Kali Linux and Windows 11 - to use the same Host-Only network adapter. They will share a private IP range and can talk to each other freely. Kali will be your attack machine. Windows 11 will be your target.

### Step 1 - Open the Virtual Network Editor in VMware

In VMware Workstation Pro, click **Edit** in the top menu bar, then click **Virtual Network Editor**.

A window will open showing a list of virtual networks. You will see entries like:

```
VMnet0    Bridged
VMnet1    Host-only    192.168.x.x   (some IP range)
VMnet8    NAT          192.168.y.y   (some IP range)
```

**VMnet1** is the default Host-Only network. It already exists. You will use this one. Take note of the subnet IP shown next to VMnet1 - it will be something like `192.168.206.0` or similar. Write down the first three parts of this IP (like `192.168.206`). You will use this later to verify connectivity.

If the subnet IP under VMnet1 is not shown, click on the VMnet1 row and look at the bottom of the editor window - it will show the subnet IP and subnet mask there.

Close the Virtual Network Editor.

### Step 2 - Set Windows 11 VM to Host-Only

Make sure your Windows 11 VM is **powered off** before changing its network adapter. Power it off from inside Windows by clicking the Start menu, clicking the power icon, and selecting **Shut down**. Wait for the VM to fully stop.

In the VMware library on the left, right-click your **Windows 11 - Lab** VM and click **Settings**.

In the VM Settings window, click on **Network Adapter** in the hardware list on the left.

On the right side, you will see the network connection type. Change it from **NAT** to **Host-only**. This connects the VM to VMnet1.

Click **OK**.

### Step 3 - Set Kali VM to Host-Only as well

Do the same for your Kali Linux VM. Power it off first, then open its Settings, click Network Adapter, and change it to **Host-only**. Click OK.

### Step 4 - Power on both VMs

Start your Windows 11 VM first. Log in with `labadmin`. Open **Command Prompt** (search for `cmd` in the Start menu and press Enter).

Type this command to find the Windows VM's IP address:

```cmd
ipconfig
```

Look in the output for the section called **Ethernet adapter Ethernet0** (or similar). Under it, look for the line that says **IPv4 Address**. It will show an IP like `192.168.206.128` or similar. Write this down. This is the Windows VM's IP address on the lab network.

### Step 5 - Test connectivity from Kali

Now start your Kali Linux VM. Open a terminal in Kali.

Type this command (replace the IP with the actual IP you got from ipconfig in the previous step):

```bash
ping 192.168.206.128
```

You should see output like this:

```
PING 192.168.206.128 (192.168.206.128) 56(84) bytes of data.
64 bytes from 192.168.206.128: icmp_seq=1 ttl=128 time=0.543 ms
64 bytes from 192.168.206.128: icmp_seq=2 ttl=128 time=0.412 ms
```

If you see replies like this, Kali can reach Windows. Your lab network is working.

If the ping times out or shows "Destination Host Unreachable", it means Windows Firewall is blocking ICMP. This is fine for now - we will confirm connectivity more thoroughly when we run actual tools. The network adapter is set correctly.

> **Note on internet access:** When both VMs are on Host-Only, they lose internet access. Whenever you need to download something inside a VM, temporarily switch that VM's network adapter back to NAT, download what you need, then switch it back to Host-Only. It only takes a few seconds in VM Settings.

---

## 7. Take Your First Snapshot

This is one of the most important habits you will build in this course. A snapshot saves the exact state of the entire VM - every file, every registry setting, every running process - at a specific moment in time. If anything goes wrong later, you can restore the VM to that saved moment instantly.

Think of it like a save point in a video game. You can break the Windows machine completely and get back to a clean state in about 30 seconds.

### Why snapshots matter so much in a lab

When you are learning privilege escalation, you will sometimes make a change to the Windows machine that you cannot reverse - maybe you accidentally delete a file needed by a service, or you modify a registry key in the wrong way, or a payload crashes the machine. Without snapshots, you would need to reinstall Windows from scratch every time. With snapshots, you just roll back.

### Step 1 - Take the first snapshot

Make sure your Windows 11 VM is running and you are logged in. Do not take snapshots while the VM is powered off - take them while it is running so that everything including RAM state is saved.

In VMware, while the Windows 11 VM is the active VM, click **VM** in the top menu bar, then hover over **Snapshot** and click **Take Snapshot...**

A dialog box will appear asking for:

- **Name:** Type exactly this:
  ```
  Clean Install - No Tools
  ```
- **Description:** You can leave this empty, or type a short note like "Fresh Windows 11 install, VMware Tools installed, no configuration done yet"

Click **Take Snapshot**. A progress bar will appear and disappear within a few seconds. The snapshot is saved.

### When to take snapshots and when to restore them

**Take a new snapshot:**
- After finishing every major configuration step (Defender disabled, user created, all tools installed)
- Before trying a new attack or technique that could mess up the system
- Whenever the machine is in a clean, known good state that you might want to return to

**Restore a snapshot:**
- When something breaks and you are not sure how to fix it
- Before starting a new lab exercise (restore to "All Tools Installed" so the machine is fresh)
- Any time you want to undo everything you have done and start a topic fresh

**How to restore a snapshot:**
In VMware, click **VM** > **Snapshot** > **Snapshot Manager**. You will see a tree of all your saved snapshots. Click on the one you want to restore and click **Go To**. VMware will ask you to confirm. Click **Go To**. The VM will revert in about 10–30 seconds.

---

## 8. Disable Windows Defender

Windows Defender is Microsoft's built-in antivirus and security tool. In a real engagement you absolutely do not want it disabled - you want to work around it. But for this course, we are learning the techniques themselves first. Running WinPEAS, GodPotato, or a custom DLL with Defender running will result in constant false positives that make learning impossible.

Once you understand each technique, you can revisit it with Defender enabled to practice evasion. That comes later.

### Why you must disable Tamper Protection first

Windows 11 has a feature called Tamper Protection. When it is on, Defender actively blocks any attempt to disable its own settings - it will ignore registry changes, Group Policy changes, and PowerShell commands that try to turn it off. You have to disable Tamper Protection through the Windows Security GUI before anything else will work.

### Step 1 - Open Windows Security

Inside the Windows 11 VM, click the **Start** button (the Windows logo in the center of the taskbar), then type **Windows Security** and press Enter.

The Windows Security app will open. It has a dark left sidebar with icons for different security features.

### Step 2 - Go to Virus and Threat Protection

In the Windows Security window, click **Virus & threat protection** in the left sidebar (it has a shield icon).

On the right side you will see a section called **Virus & threat protection settings**. Underneath it, click the link that says **Manage settings**.

### Step 3 - Turn off Tamper Protection

Scroll down on this settings page until you find **Tamper Protection**. It will have a toggle switch that is currently set to **On** (blue).

Click the toggle to turn it **Off**. A User Account Control (UAC) prompt will appear asking "Do you want to allow this app to make changes to your device?" Click **Yes**.

The toggle will now show **Off**.

### Step 4 - Turn off Real-time Protection

Now scroll back up to the top of this same settings page. The first option is **Real-time protection**. Its toggle should also be **On** (blue). Click it to turn it **Off**.

Another UAC prompt may appear. Click **Yes**.

The toggle will now show **Off** and the Windows Security page will show a yellow warning triangle saying "Your device may be vulnerable." That is expected and fine for a lab.

### Step 5 - Disable Defender permanently via Group Policy

The Real-time protection toggle will turn itself back on after a reboot because Windows automatically re-enables it. To make the disable permanent, you need to use the Group Policy Editor.

Press **Windows + R** on the keyboard to open the Run dialog. Type `gpedit.msc` and press Enter.

The Local Group Policy Editor will open. In the left panel, expand the folders in this exact order:

```
Computer Configuration
  └── Administrative Templates
        └── Windows Components
              └── Microsoft Defender Antivirus
```

Click on the **Microsoft Defender Antivirus** folder to select it. In the right panel, you will see a list of policies. Double-click on the one called **Turn off Microsoft Defender Antivirus**.

A settings dialog will open. Select the **Enabled** radio button (yes, "Enabled" here means "yes, enable the setting" which means "yes, turn off Defender"). Click **Apply**, then click **OK**.

### Step 6 - Confirm Defender is off

Open **Task Manager** (press Ctrl+Shift+Esc), click on the **Details** tab, and look for a process called `MsMpEng.exe` (this is the Defender engine). If you do not see it in the list, Defender is not running.

Alternatively, open the Windows Security app again. You will see warning messages about things being turned off. That is the confirmation.

---

## 9. Disable Windows Automatic Updates

Windows 11 will try to update itself and restart at inconvenient times. More importantly, an update can change the state of your lab machine - it might patch a vulnerability you were about to practice, or change how a feature behaves. Keeping the lab in a stable, known state means stopping updates.

### Step 1 - Disable the Windows Update service

Press **Windows + R**, type `services.msc`, and press Enter.

The Services window will open showing a long list of Windows services. Scroll down (the list is alphabetical) until you find **Windows Update**.

Right-click on **Windows Update** and click **Properties**.

In the Properties window:

1. Under **Service status**, click the **Stop** button to stop the service immediately
2. Change the **Startup type** dropdown from **Manual** to **Disabled**
3. Click **Apply**, then click **OK**

Now find **Windows Update Medic Service** in the list (it may also be called **WaaSMedicSvc**). Right-click it and try to open Properties. You may find that this service cannot be disabled through the normal method - Microsoft protects it. If it shows grayed out options, that is fine. We will handle it through Group Policy.

### Step 2 - Disable updates via Group Policy as backup

Open the Group Policy Editor again: press **Windows + R**, type `gpedit.msc`, press Enter.

Navigate to:

```
Computer Configuration
  └── Administrative Templates
        └── Windows Components
              └── Windows Update
                    └── Manage end user experience
```

In the right panel, double-click **Configure Automatic Updates**.

In the settings dialog, select **Disabled**. Click **Apply**, then **OK**.

Also go back one level to the **Windows Update** folder (not the subfolder) and double-click **Remove access to use all Windows Update features**. Set this to **Enabled**, then click **Apply** and **OK**.

### Step 3 - Confirm updates are stopped

Press **Windows + I** to open Windows Settings. Click on **Windows Update** in the left sidebar. You should see a message saying updates are paused or that your organization manages Windows Update settings. If you see that, it worked.

---

## 10. Create a Low-Privilege User for Practice

Every privilege escalation technique in this course starts from a low-privilege account. You need a user that is NOT an administrator. This is `labuser` - the account you will use for all the labs.

### Step 1 - Open Computer Management

Right-click on the **Start button** (the Windows logo in the taskbar) and click **Computer Management** from the context menu that appears.

Computer Management will open. It has a tree of categories on the left side.

### Step 2 - Navigate to Local Users

In the left panel, expand **Local Users and Groups** by clicking the arrow next to it. Then click on **Users**.

In the center panel, you will see the existing user accounts - you should see at least **labadmin** (the account you created during setup), **Administrator**, and **Guest**.

### Step 3 - Create the new user

Right-click anywhere in the empty space in the center panel (or right-click on the **Users** folder in the left panel) and click **New User...**.

A dialog box will appear. Fill it in exactly like this:

| Field | Value |
|---|---|
| User name | `labuser` |
| Full name | `Lab User` |
| Description | `Low privilege practice account` |
| Password | `Password123` |
| Confirm password | `Password123` |

Uncheck the box that says **User must change password at next logon**.

Leave **User cannot change password** and **Account is disabled** unchecked.

Click **Create**, then click **Close**.

### Step 4 - Confirm the user is NOT an administrator

In the Users list, double-click on **labuser** to open its properties.

Click on the **Member Of** tab. You should see only one group listed: **Users**. There should be no **Administrators** group in this list.

If you see Administrators listed, click on it, then click the **Remove** button.

Click **Apply** and **OK**.

### Step 5 - Test the new account

Let us confirm the account works. Click the **Start** button, click on the user icon (top area of Start menu shows `labadmin`), and click **Sign out**.

At the Windows lock screen, click on **Other user** or click on **labuser** if it appears. Log in with:

```
Username: labuser
Password: Password123
```

Windows will set up the user profile for the first time - this takes about 30 seconds and then you will land on the desktop.

Open a Command Prompt and run:

```cmd
whoami /groups
```

Look at the output. You should see that `labuser` is a member of the **Users** group and **BUILTIN\Users**, but NOT a member of **Administrators** or **BUILTIN\Administrators**. This confirms the account has low privileges.

Sign back out and log back in as `labadmin` for the next steps.

---

## 11. Take Your Second Snapshot

The machine is now configured the way we want it for all labs. Before loading any tools, save this state.

In VMware, click **VM** > **Snapshot** > **Take Snapshot...**

Fill in:

- **Name:**
  ```
  Lab Ready - Defender Off - Low Priv User Created
  ```
- **Description:** Optional. You can write: "Windows Defender disabled permanently, auto-updates stopped, labuser account created with no admin rights"

Click **Take Snapshot**.

This is your base snapshot. If you ever want to get back to a clean Windows install with Defender off and the user accounts ready but no tools installed, restore this snapshot.

---

## 12. Install All Tools on the Windows VM

All tools go in a folder called `C:\Tools`. This keeps everything organized and easy to find. Open **File Explorer**, navigate to the C: drive, right-click in the empty space, click **New** > **Folder**, and name it `Tools`.

Inside `Tools`, create two more folders:

```
C:\Tools\SysTools     (for system analysis and debugger tools)
C:\Tools\RedTeam      (for offensive tools)
```

---

### Process Hacker 2

**What it is and why you need it:**
Process Hacker 2 is the single most useful GUI tool for understanding what Windows is doing under the hood. It shows you every running process, every thread inside each process, what handles each process has open, what DLLs are loaded, and - most importantly for this course - it shows you the exact security token and privileges of each process. It is what you will have open during the token impersonation and DLL hijacking labs to watch things happen in real time. Note that the newer version is called "System Informer" but Process Hacker 2 is the classic version that most security guides still reference.

**Download:**
Go to this address in the browser inside your Windows VM:

```
https://sourceforge.net/projects/processhacker/files/processhacker2/processhacker-2.39-setup.exe/download
```

Alternatively search for **Process Hacker SourceForge** and click the green Download button on the project page. The file is called `processhacker-2.39-setup.exe`.

**Installation:**
1. Double-click the downloaded `processhacker-2.39-setup.exe`
2. Click **Next** through the wizard
3. Accept the license agreement and click **Next**
4. Leave the install location as default and click **Next**
5. Click **Install**, then **Finish**

**Confirming it works:**
Search for **Process Hacker** in the Start menu and run it. You should see a window listing all running processes with CPU and memory usage, similar to Task Manager but with much more detail. You will learn how to use it in later files.

---

### Sysinternals Suite

**What it is and why you need it:**
Sysinternals is a collection of free tools made by Microsoft that give you deep visibility into Windows internals. The four tools you will use most in this course are:

- **Process Monitor (Procmon):** Watches every single file, registry, and network operation every process makes, in real time. You will use this to find DLL hijacking opportunities by watching which DLLs a program tries to load.
- **AccessChk:** A command-line tool that tells you what permissions a user has on files, folders, services, and registry keys. You will use this to find service misconfigurations.
- **Autoruns:** Shows every single place where code is configured to run automatically when Windows starts. You will use this for persistence analysis.
- **Process Explorer:** A better, more detailed version of Task Manager with a full process tree view.

**Download:**
Go to:

```
https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite
```

Click the **Download Sysinternals Suite** link. This downloads a ZIP file called `SysinternalsSuite.zip`. It is about 160–180 MB.

**Installation:**
Sysinternals does not need a traditional installer. Just extract the ZIP.

Right-click `SysinternalsSuite.zip` and click **Extract All...**. In the extraction dialog, change the destination path to:

```
C:\Tools\SysTools\Sysinternals
```

Click **Extract**. This will unzip all the Sysinternals tools into that folder.

**Confirming it works:**
Open File Explorer and browse to `C:\Tools\SysTools\Sysinternals`. You should see hundreds of `.exe` files. Double-click `procmon64.exe` to launch Process Monitor. It will ask if you want to agree to the license - click **Agree**. The Process Monitor window will open showing a live stream of all system activity. Press **Ctrl+E** to stop the capture. That confirms it works. Close it for now.

---

### x64dbg

**What it is and why you need it:**
x64dbg is a free debugger for Windows. A debugger lets you stop a running program at any point, inspect what is in memory, step through code one instruction at a time, and understand exactly what a program is doing internally. You will use x64dbg in later files when you need to understand how a Windows binary works, how EDR hooks look in NTDLL, and how shellcode behaves in memory.

**Download:**
Go to:

```
https://x64dbg.com
```

Click the **Download** button on the main page. This takes you to the GitHub releases page. Download the latest release ZIP file - it will be named something like `snapshot_YYYY-MM-DD_*.zip`.

**Installation:**
x64dbg does not need an installer. Right-click the downloaded ZIP and click **Extract All...**. Extract it to:

```
C:\Tools\SysTools\x64dbg
```

Inside that folder you will find a folder called `release`. Inside `release` you will find folders called `x32` and `x64`, and in each one there is an `x64dbg.exe` (or `x32dbg.exe` for the 32-bit version).

Navigate to `C:\Tools\SysTools\x64dbg\release\x64\` and double-click `x64dbg.exe`. The debugger will open showing a gray interface with menus at the top. Close it for now - you will learn how to use it in the advanced files.

**Optional step:** Right-click `x64dbg.exe`, click **Create shortcut**, and put the shortcut on the Desktop for easy access.

---

### WinDbg

**What it is and why you need it:**
WinDbg is Microsoft's official debugger. Unlike x64dbg which debugs user-mode programs, WinDbg can debug the Windows kernel itself - meaning you can stop the entire operating system at a breakpoint and inspect kernel structures. You will use WinDbg in this course to look at kernel objects like EPROCESS blocks and access tokens as they actually exist in kernel memory. This is how you build real understanding, not just surface knowledge.

**Download and Installation:**
WinDbg is installed from the Microsoft Store, which is the easiest way to get it.

Press the **Windows + S** keys (or click the search bar in the taskbar) and type **Microsoft Store**. Click on the Microsoft Store app to open it.

Inside the Store, click the search bar at the top and type **WinDbg**. Look for the result called **WinDbg** published by **Microsoft Corporation**. Click on it.

Click the **Install** or **Get** button. The Store will download and install WinDbg automatically. This takes a few minutes.

When done, search for **WinDbg** in the Start menu and run it to confirm it opens. You will see a modern-looking debugger interface. Close it for now.

---

### Why There Is No C Compiler on the Windows VM

You might expect to see Visual Studio or a C compiler in this list, since this course teaches you to write DLLs and payloads in C. There is a deliberate reason neither is installed here: **you would never install a compiler on a machine you have compromised.**

Think about how a real engagement works. You get code execution on a Windows machine as a low-privilege user. You find a DLL hijacking opportunity. The next step is to write a malicious DLL and drop it in the right folder. In a real engagement, you write and compile that DLL on your own attack machine - your Kali box - before it ever goes anywhere near the target. You compile it, transfer the finished `.dll` file over HTTP or SMB, and drop it on the machine.

Installing a compiler on a machine you have compromised would be a glaring red flag. It would show up in Windows event logs, in EDR telemetry, in Defender alerts, and in any forensic investigation that happens afterwards. Investigators looking at a compromised machine and seeing Visual Studio installed, or mingw, or any compiler toolchain, immediately know someone was developing offensive tools there. That is not something any real red team operator would leave behind.

The rule is straightforward: **all compilation happens on your attack machine, offline, before or during the engagement. The target machine only ever sees the finished compiled binary.**

For this course, Kali Linux is your attack machine. All C code - the DLLs in file 14 and the syscall code in file 22 - will be compiled on Kali using a tool called **mingw-w64**, which cross-compiles Windows binaries from Linux. The compiled `.exe` or `.dll` is then transferred to the Windows VM using a Python HTTP server on Kali and downloaded with PowerShell on the Windows side. This is exactly how it works on real engagements, and the full setup for that is in the next section.

---

## 13. Download Red Team Tools to C:\Tools\RedTeam

These tools do not need to be installed - they are standalone executables. You download them, place them in `C:\Tools\RedTeam`, and they are ready to use. Since Defender is disabled, they will not be deleted.

For each tool, download it inside the Windows VM, save it to `C:\Tools\RedTeam`, and do a quick sanity check.

---

### WinPEAS

**What it does:** Automatically scans a Windows machine for hundreds of privilege escalation paths - service misconfigs, token privileges, stored credentials, weak file permissions, misconfigured registry keys, and more. It is your first tool to run after getting a shell on a new machine.

**Download:**
Go to this GitHub releases page:

```
https://github.com/peass-ng/PEASS-ng/releases
```

Click on the latest release at the top. Scroll through the assets and download the file called **`winPEASany_ofs.exe`** - this is the obfuscated version that works on all Windows architectures.

Save it to `C:\Tools\RedTeam\winPEASany.exe`.

**Quick test:**
Open a Command Prompt, navigate to `C:\Tools\RedTeam`, and run:

```cmd
cd C:\Tools\RedTeam
winPEASany.exe --help
```

You should see a help message with usage options. That confirms it runs.

---

### Seatbelt

**What it does:** Does deep host reconnaissance - it collects information about the machine in a structured way. While WinPEAS looks for privesc paths, Seatbelt collects operational security information: what AV is running, what processes are active, what network connections exist, what credentials might be cached, what PowerShell history exists. You use it to profile a machine thoroughly.

**Download:**
Go to:

```
https://github.com/GhostPack/Seatbelt/releases
```

Download the file called `Seatbelt.exe` from the latest release assets.

Save it to `C:\Tools\RedTeam\Seatbelt.exe`.

**Quick test:**

```cmd
cd C:\Tools\RedTeam
Seatbelt.exe -group=all 2>&1 | more
```

You should see output scrolling with various system information sections. Press `Q` to quit the `more` pager. That confirms it works.

---

### PrivescCheck

**What it does:** A PowerShell script that checks for privilege escalation opportunities with clean, structured output. Particularly good at finding service misconfigurations and scheduled task weaknesses. Produces results that are easy to read compared to WinPEAS.

**Download:**
Go to:

```
https://github.com/itm4n/PrivescCheck/releases
```

Download the file called `PrivescCheck.ps1` from the latest release.

Save it to `C:\Tools\RedTeam\PrivescCheck.ps1`.

**Quick test:**
Open a PowerShell window as Administrator (search PowerShell, right-click, Run as administrator). Run:

```powershell
Set-ExecutionPolicy Bypass -Scope Process
cd C:\Tools\RedTeam
.\PrivescCheck.ps1
```

You will see colored output listing the machine's security configuration and any findings. Let it run to completion (takes 1–2 minutes).

---

### GodPotato

**What it does:** A token impersonation tool. When a service account has SeImpersonatePrivilege, GodPotato uses a DCOM activation trick to force the SYSTEM account to authenticate to a local named pipe controlled by GodPotato. It captures the SYSTEM token and uses it to run whatever command you specify as SYSTEM. Works on Windows Server 2012 through Windows 11 and Server 2022.

**Download:**
Go to:

```
https://github.com/BeichenDream/GodPotato/releases
```

Download the file called `GodPotato-NET4.exe` (the .NET 4 version works on the widest range of machines).

Save it to `C:\Tools\RedTeam\GodPotato.exe`.

**Quick test - do NOT run this yet:**
We will use GodPotato properly in file 13. For now, just confirm it is there:

```cmd
dir C:\Tools\RedTeam\GodPotato.exe
```

---

### PrintSpoofer

**What it does:** Another token impersonation tool, but it uses a different technique involving the Windows Print Spooler service and named pipes. Works well on Windows 10 and Windows Server 2016/2019 when the DCOM method used by GodPotato is blocked or restricted.

**Download:**
Go to:

```
https://github.com/itm4n/PrintSpoofer/releases
```

Download the file called `PrintSpoofer64.exe`.

Save it to `C:\Tools\RedTeam\PrintSpoofer64.exe`.

---

### SigmaPotato

**What it does:** The newest and most versatile token impersonation tool. It is a fork of GodPotato with two important improvements: it supports a wider OS range (Windows 8 through Windows 11, Server 2012 through 2022), and it can run entirely in memory using .NET reflection - meaning the binary never needs to touch disk. This makes it significantly harder for EDR to detect compared to the others.

**Download:**
Go to:

```
https://github.com/tylerdotrar/SigmaPotato/releases
```

Download the file called `SigmaPotato.exe`.

Save it to `C:\Tools\RedTeam\SigmaPotato.exe`.

---

### Final check on the RedTeam folder

Open Command Prompt and run:

```cmd
dir C:\Tools\RedTeam
```

You should see all six tools listed:

```
winPEASany.exe
Seatbelt.exe
PrivescCheck.ps1
GodPotato.exe
PrintSpoofer64.exe
SigmaPotato.exe
```

If any are missing, download them now before moving on.

---

## 14. Set Up C Compilation on Kali

This section sets up your Kali machine as the place where all C code gets compiled. You will do this work on Kali, not inside the Windows VM. Once you understand why, the whole workflow will make sense every time you use it.

### What mingw-w64 is and how cross-compilation works

Normally, if you want to compile a Windows `.exe` or `.dll`, you need to be on a Windows machine with a Windows compiler. But there is a tool called **mingw-w64** (short for Minimalist GNU for Windows, 64-bit) that runs on Linux and produces Windows binaries. You write your C code on Kali, run the mingw compiler, and out comes a `.exe` or `.dll` that runs on Windows. The Linux machine never has to know it is making Windows files - it just calls a different compiler.

This is called **cross-compilation**. You are compiling for a different operating system than the one you are running on.

The reason red teamers use this: your attack machine is Kali. Your target is Windows. You compile on Kali, transfer the binary to the Windows target, and execute it there. Kali never touches the target's disk until the finished payload arrives. No compiler, no build tools, no source code ever lands on the target.

### Step 1 - Install mingw-w64 on Kali

Open a terminal on your Kali machine and run:

```bash
sudo apt update
sudo apt install mingw-w64 -y
```

This installs the full mingw-w64 toolchain, which includes compilers for both 32-bit and 64-bit Windows targets. The package is about 100–200 MB and takes a few minutes to download.

When it is done, confirm it installed correctly by checking the version:

```bash
x86_64-w64-mingw32-gcc --version
```

You should see output like:

```
x86_64-w64-mingw32-gcc (GCC) 13.x.x
Copyright (C) 2023 Free Software Foundation, Inc.
```

That is the 64-bit Windows cross-compiler. There is also a 32-bit version:

```bash
i686-w64-mingw32-gcc --version
```

For almost everything in this course you will use the 64-bit one (`x86_64-w64-mingw32-gcc`) since modern Windows machines are 64-bit.

### Step 2 - Compile a Windows .exe from Kali (Hello World test)

Let us confirm the compiler works end to end by building a simple Windows executable on Kali and running it on the Windows VM.

On Kali, create a test file:

```bash
nano /tmp/hello.c
```

Type this into the file (this is your first C program - do not worry about understanding every part yet, that comes in file 14):

```c
#include <stdio.h>
#include <windows.h>

int main() {
    MessageBoxA(NULL, "Hello from Kali!", "Test", MB_OK);
    return 0;
}
```

Save and exit nano: press **Ctrl+X**, then **Y**, then **Enter**.

Now compile it into a Windows 64-bit executable:

```bash
x86_64-w64-mingw32-gcc /tmp/hello.c -o /tmp/hello.exe -luser32
```

Breaking this command down so you understand each part:

| Part | What it means |
|---|---|
| `x86_64-w64-mingw32-gcc` | The cross-compiler for 64-bit Windows |
| `/tmp/hello.c` | Your source file (input) |
| `-o /tmp/hello.exe` | The output file name and location |
| `-luser32` | Link against the `user32.dll` library (needed for MessageBoxA) |

Check that the file was created:

```bash
ls -lh /tmp/hello.exe
```

You should see something like `-rwxr-xr-x 1 kali kali 72K ... hello.exe`. The file exists. Now transfer it to the Windows VM and double-click it - you will see a popup saying "Hello from Kali!". That confirms your entire compilation pipeline works.

How to transfer the file is explained in Step 4.

### Step 3 - Compile a Windows .dll from Kali

A DLL uses a slightly different compiler flag. The key difference is `-shared`, which tells the compiler to produce a DLL instead of a standalone executable.

Create a test DLL source file:

```bash
nano /tmp/test.c
```

Type this:

```c
#include <windows.h>

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) {
    if (fdwReason == DLL_PROCESS_ATTACH) {
        MessageBoxA(NULL, "DLL loaded!", "Test DLL", MB_OK);
    }
    return TRUE;
}
```

Save and exit. Now compile it as a DLL:

```bash
x86_64-w64-mingw32-gcc -shared -o /tmp/test.dll /tmp/test.c -luser32
```

The only difference from compiling an `.exe` is the `-shared` flag. Check it was created:

```bash
ls -lh /tmp/test.dll
```

You will use this exact command structure in file 14 when building the actual malicious DLL for the hijacking lab. The code inside will be different, but the compile command will look exactly like this.

### Step 4 - Transfer compiled files from Kali to the Windows VM

The standard way to move a file from Kali to a Windows target during an engagement is a Python HTTP server. You spin up a quick web server on Kali, then use PowerShell on the Windows side to download the file. This same technique works in the real world and in your lab.

**On Kali - start the HTTP server:**

Navigate to the folder containing your compiled file and start the server:

```bash
cd /tmp
python3 -m http.server 8080
```

You should see:

```
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Your Kali machine is now serving everything in `/tmp` over HTTP on port 8080. Leave this terminal open. The server keeps running as long as this terminal is open.

**Find your Kali IP address:**

Open a second terminal on Kali and run:

```bash
ip addr show eth0
```

Look for the `inet` line. The IP will be something like `192.168.206.129`. This is the IP of your Kali machine on the lab network.

**On the Windows VM - download the file using PowerShell:**

Open PowerShell on the Windows VM (search for it in the Start menu). Run this command, replacing the IP with your actual Kali IP:

```powershell
Invoke-WebRequest -Uri "http://192.168.206.129:8080/hello.exe" -OutFile "C:\Tools\RedTeam\hello.exe"
```

You can also use this shorter version:

```powershell
iwr "http://192.168.206.129:8080/hello.exe" -OutFile "C:\Tools\RedTeam\hello.exe"
```

`iwr` is the short alias for `Invoke-WebRequest` in PowerShell. Both commands do exactly the same thing: download the file from the URL and save it to the path you specify.

After running the command, check that the file arrived:

```powershell
dir C:\Tools\RedTeam\hello.exe
```

If it is there, double-click it in File Explorer. The popup saying "Hello from Kali!" will appear. You have successfully compiled a Windows binary on Kali and run it on Windows.

**On Kali - stop the HTTP server:**

Go back to the terminal where the HTTP server is running and press **Ctrl+C** to stop it.

> **This workflow - compile on Kali, serve with Python HTTP, download with PowerShell - is the workflow for every C program and DLL in this course. It is also the workflow used on real engagements. Learn it now and it will feel automatic by the time you need it in the attack files.**

---

## 15. Take the Final Snapshot

Both your Windows VM and your Kali machine are now fully configured. The Windows VM has all the analysis and offensive tools installed. Kali has the C compilation toolchain set up. This is the state you want to preserve as your starting point for every lab in this course.

Before taking this snapshot, confirm both machines are ready:

**On the Windows VM**, make sure:
- `C:\Tools\SysTools\` has Process Hacker 2, Sysinternals, x64dbg, and WinDbg
- `C:\Tools\RedTeam\` has all six offensive tools
- Defender is off, updates are disabled, `labuser` exists

**On Kali**, make sure:
- `x86_64-w64-mingw32-gcc --version` returns a version number
- You can start a Python HTTP server with `python3 -m http.server 8080`

When both are confirmed, go to VMware, click **VM** > **Snapshot** > **Take Snapshot...**

Fill in:

- **Name:**
  ```
  All Tools Installed - Ready for Course
  ```
- **Description:** "Windows: all tools installed, Defender off, updates off, labuser created. Kali: mingw-w64 installed, file transfer method tested."

Click **Take Snapshot**.

You should now have three snapshots in total:

1. **Clean Install - No Tools** - bare Windows 11, nothing configured
2. **Lab Ready - Defender Off - Low Priv User Created** - configured but no tools
3. **All Tools Installed - Ready for Course** - your main working snapshot

---

## 16. How to Use Snapshots Throughout This Course

Here is the exact workflow for every lab in this course:

**Before starting any new lab:**
1. In VMware, go to **VM** > **Snapshot** > **Snapshot Manager**
2. Click on **All Tools Installed - Ready for Course**
3. Click **Go To** and confirm
4. The VM reverts to the clean state with all tools ready
5. Log in as `labuser` (the low-privilege account) to start the lab
6. Switch to `labadmin` when you need to set up the vulnerable condition that you will then exploit as `labuser`

**When something breaks:**
Just restore the snapshot. Do not try to manually fix whatever went wrong - restoring takes 20 seconds and puts you back in a known good state. The only thing you lose is time, and snapshots mean you never lose much of it.

**When you finish a lab you want to keep:**
Before restoring for the next lab, take a snapshot first and name it something descriptive like "After Token Impersonation Lab - 13". That way you can go back and review your work. You can always delete old snapshots later to free disk space.

---

## 17. Summary of What You Did in This File

Here is everything you set up in this file, in order:

1. Downloaded Windows 11 Enterprise Evaluation ISO (free, 90-day trial, from Microsoft)
2. Created a new VMware VM with the correct specs - 8 GB RAM, 4 CPU cores, 80 GB disk, UEFI firmware
3. Added a virtual TPM 2.0 by encrypting the VM and adding a TPM device through VMware settings
4. Installed Windows 11 and created a local admin account called `labadmin` without a Microsoft account
5. Installed VMware Tools for proper screen resolution and mouse integration
6. Set up a Host-Only network so Kali and Windows can communicate privately
7. Disabled Windows Defender - first turned off Tamper Protection, then Real-time Protection, then used Group Policy for a permanent disable
8. Disabled Windows automatic updates via Services and Group Policy
9. Created `labuser` - a standard user with no admin rights - for privilege escalation practice
10. Installed Process Hacker 2, Sysinternals Suite, x64dbg, and WinDbg on the Windows VM
11. Downloaded all six offensive tools into `C:\Tools\RedTeam`
12. Installed mingw-w64 on Kali, compiled a test `.exe` and `.dll` from C source on Kali, transferred them to Windows using a Python HTTP server and PowerShell
13. Saved three named snapshots as restore points

Your lab is ready. Every technique in this course can now be practiced safely, reset instantly, and repeated as many times as needed.

---

## 18. Interview Questions

---

### Theory Questions

**Question 1: Why do red teamers use virtual machines for practice instead of real machines?**

**Full answer:** Virtual machines let you set up a completely controlled, isolated environment that you can reset instantly if something goes wrong. A real machine is your actual system - if you break it during practice, you have a real problem on your hands. A VM is a file on your hard drive. If you mess it up, you delete it or restore a snapshot and you are back to a clean state in 30 seconds.

There is also the isolation benefit. When you practice exploitation techniques on a VM on a Host-Only network, none of that activity touches the real network. You can run tools, exploit services, and make as much noise as you want without any risk of affecting other machines or triggering real alerts.

**How to say this confidently in an interview:** "VMs are fundamental to lab work because they give you snapshots - a clean rollback point. You can break the machine, learn from what happened, restore, and try again. The isolation also matters - a Host-Only network means none of your practice traffic leaves your machine."

---

**Question 2: What is the difference between NAT and Host-Only networking in VMware, and when do you use each?**

**Full answer:** NAT (Network Address Translation) gives the VM internet access through the host machine. The VM has a private IP that VMware translates to the host's real IP when traffic goes out. The VM can reach the internet, but the internet cannot directly reach the VM. You use NAT when you need a VM to download files or access the internet.

Host-Only creates a private network that is completely isolated inside VMware. VMs on the same Host-Only network can talk to each other and to the host machine, but nothing gets in or out to the real internet or real network. You use Host-Only when you want two VMs to communicate - like a Kali attacker and a Windows target - without any risk of traffic leaking out.

For a red team lab, you want Host-Only during practice so your attack traffic stays inside your machine.

**How to say this confidently in an interview:** "NAT is for internet access, Host-Only is for isolated inter-VM communication. In a lab, you run Host-Only so attack traffic never leaves the machine. In a real engagement, you'd have a different network consideration entirely - you're on the client's actual network."

---

**Question 3: Why do we disable Windows Defender on the lab machine, and would you do this on a real engagement?**

**Full answer:** In the lab, Defender is disabled because we are learning the techniques themselves, not the evasion. When you are practicing privilege escalation for the first time, having Defender constantly delete your tools and flag your actions prevents you from understanding what is actually happening. You disable it so you can see the technique work cleanly, understand every step, and build a mental model of what is happening.

On a real engagement, you absolutely do not disable Defender - that is not the goal and you usually cannot anyway. On a real engagement, bypassing AV and EDR detection is part of the skill. Some of the later files in this course (AMSI bypass, ETW patching, direct syscalls) teach exactly how to do that. But you need to understand the underlying technique first before you can understand what evasion is trying to achieve.

**How to say this confidently in an interview:** "We disable it in the lab to learn the technique cleanly - you need to understand the attack before you can understand how to make it stealthy. On a real engagement, you work within the EDR constraints and use evasion techniques. The evasion files later in this course cover that. Defender gets disabled in the lab the same way a piano student slows down to 50% speed to learn a piece - then speeds it up with all constraints in place."

---

### Scenario-Based Questions

**Question 4: You are on a real engagement and you need to test a new privilege escalation technique. Your test VM got corrupted and will not boot. What do you do?**

**Full answer:** This is exactly why snapshots exist and why you should be religious about taking them. If the VM will not boot and you have a snapshot, restore the snapshot - 30 seconds and you are back. If you somehow do not have a snapshot, you have two options: rebuild the VM from scratch (which takes about an hour with this lab setup file as a reference), or if you have a working backup of the VM folder, copy the folder back.

The lesson from this scenario is to always take a snapshot before trying anything destructive. The habit is: before every new lab, restore to your base snapshot. Before trying something experimental, take a new snapshot. That way you are never more than one snap away from a clean state.

In a real engagement (not just a lab), this scenario translates to something different: you would not be testing new techniques on client systems. You test in your own lab first, understand the technique completely, then use it on the engagement. If something unexpected happens on the client machine, you document it and report it, you do not try to clean it up yourself.

**How to say this confidently in an interview:** "This is what snapshots solve. Restore the snapshot, you're back in 30 seconds. If there's no snapshot, rebuild - which is why I take snapshots before anything experimental. On a real engagement, you test techniques in your own lab before running them on client systems."

---

**Question 5: During an engagement, you get code execution as a low-privilege user on a Windows server. The client has told you that their security team monitors for tool executions. What is your first step and why?**

**Full answer:** The first step is not to run any tools at all. Before running anything that could trigger an alert, you do manual reconnaissance using only Windows built-in commands. This is exactly what file 10 in this course covers.

The very first command is `whoami /priv` - just to see what privileges the current token has. This is a safe, built-in Windows command that no security team monitors specifically. If SeImpersonatePrivilege shows up, you know a potato attack will work and you plan from there.

Then you do a slow, careful manual sweep: check services with `sc qc`, check folder permissions with `icacls`, check scheduled tasks with `schtasks /query`, check registry keys with `reg query`. All built-in Windows commands. None of these trigger EDR behavioral rules. You build a picture of the machine manually before touching any offensive tool.

Only after you have identified a specific opportunity do you introduce a tool - and even then, you try to use the simplest, least detectable approach first.

**How to say this confidently in an interview:** "Manual first. `whoami /priv` is the first command - it's built-in, tells you the token privilege immediately, and nothing logs it specially. Then slow manual recon with built-in commands. You understand the machine before running anything that could generate an alert. Tools come in only when you have a specific target in mind and you have picked the cleanest approach."

---

**Question 6: You set up a Windows 11 VM for a client to use as a phishing simulation target, but when you try to run your test payload on it, Windows Defender immediately deletes it. What are your options?**

**Full answer:** If this is a controlled lab scenario where Defender should be off, go back through the Defender disable steps from this file. Check that Tamper Protection is off, Real-time Protection is off, and the Group Policy setting for turning off Defender is set to Enabled. All three need to be in place because Defender can re-enable itself if only one or two of those steps are done.

If this is a real engagement scenario where Defender should be running (because you are testing your payload against actual defenses), then Defender deleting your payload is telling you something important: your payload needs better evasion. This is actually useful feedback. You need to modify the payload to bypass AMSI (if it is a script), change the file content to avoid signature detection (if it is a binary), or use a loader that uses in-memory execution so the payload never touches disk at all.

The third option - and what most professional red teamers do - is to use techniques like custom shellcode loaders, DLL injection into trusted processes, or direct syscalls so that the payload never looks like a payload to Defender's eyes. Those are the techniques covered in files 20 through 22 of this course.

**How to say this confidently in an interview:** "If it's the lab machine where Defender should be off, walk through the three-step disable process - Tamper Protection off, Real-time Protection off, Group Policy setting applied. If it's a real engagement where Defender is supposed to be running, then Defender catching the payload means the payload needs evasion work - AMSI bypass, in-memory execution, or a custom loader. The catch is the feedback."
