---
layout: post
title: "How to install Ubuntu 20.04 LTS on Oracle VM VirtualBox"
description: "Install Ubuntu 20.04 on Oracle VM VirtualBox step by step."
crawlertitle: "How to install Ubuntu 20.04 LTS on Oracle VM VirtualBox"
date: 2020-07-16 21:15:49 +0800
categories: Linux
tags: Linux
comments: true
---

- toc
{:toc}

---
## Install Ubuntu 20.04 LST on Oracle VM VirtualBox
- Download the desired version of Ubuntu image from the official site:  
  [http://ftp.ubuntu-tw.org/mirror/ubuntu-releases/](http://ftp.ubuntu-tw.org/mirror/ubuntu-releases/)
- Create a new VM:  
  ![](https://i.imgur.com/EFDd4cq.png)

  ![](https://i.imgur.com/eoRzAoL.png)  

  ![](https://i.imgur.com/DGy9NUE.png)

  ![](https://i.imgur.com/DeXCWpr.png)

  ![](https://i.imgur.com/bqrkJjJ.png)

- Settings of VM:
  - Modify boot order:
    ![](https://i.imgur.com/TKFQPPr.png)

  - General:
    ![](https://i.imgur.com/i65sOrx.png)

  - System:
    ![](https://i.imgur.com/mx0E3jw.png)

  - Display:
    Maximize the Video Memory to avoid black screen when booting into black screen.
    ![](https://i.imgur.com/PO4je0P.png)

  - Storage:
    - Select Optical Drive > Choose the virtual optical disk file which downloaded at first step.
  ![](https://i.imgur.com/qaSYAKG.png)

  - Customize Shared Folder:
    To share files between your host system and virtual machine.
    ![](https://i.imgur.com/pJjc7ug.png)

#### Start the virtual machine and install Ubuntu through virtual optical disk
![](https://i.imgur.com/nmlyzaG.png)

![](https://i.imgur.com/lgNfee0.png)

![](https://i.imgur.com/Rk8t5Du.png)

Press ENTER to remove the medium and continue booting.
![](https://i.imgur.com/pFSGLeH.png)

- Set up a proxy server if needed:
  e.g. 169.19.8.10:8080
  ![](https://i.imgur.com/x55grHc.png)

#### How to active the Shared Clipboard?
- Devices > Insert Guest Additions CD Image
  ![](https://i.imgur.com/VK1P53D.png)

  ![](https://i.imgur.com/CBSnqbD.png)

  ![](https://i.imgur.com/hZyQIYn.png)

##### Fix the error when inserting the Guest Additions CD Image
![](https://i.imgur.com/K14n1W8.png)

Simply unmount the existing CD image, then it would be able to insert again for the autorun.  
![](https://i.imgur.com/Iz9uS4N.png)

#### How to configure `apt` behind proxy server
- Create/Edit `apt.conf`:  
`sudo vi /etc/apt/apt.conf`  

- Insert content below in `apt.conf`, then we're all set.
  ```
  Acquire::http::Proxy "http://169.19.8.10:8080";
  ```

- Check your proxy settings: 
  ```
  $ env | grep proxy`

  # https_proxy=http://169.19.8.10:8080
  # http_proxy=http://169.19.8.10:8080
  # no_proxy=localhost,127.0.0.0/8,::1
  ```
![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)
