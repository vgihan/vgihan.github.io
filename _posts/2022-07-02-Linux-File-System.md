---
layout: post
title: Linux File System
author: vgihan
description: study Linux File System
tags: [linux, filesystem]
categories: [linux]
published: true
extensions:
  preset: gfm
---

컴퓨터에서 파일이나 자료를 쉽게 발견 및 접근할 수 있도록 보관, 조직하는 체제를 가리키는 말이라고 한다. 파일 시스템은 운영체제 별로 다를 수 있기 때문에, 여러가지 종류가 존재한다. 이번 글은 리눅스에서 사용되는 파일 시스템인 EXT에 대해 설명하고자 한다.

![image](https://user-images.githubusercontent.com/49841765/176985965-1c7266c2-b7f2-4c2a-bb24-1abfdfad7dcd.png)

### Blocks

디스크는 고정된 크기의 block으로 나누어진다. 일반적으로 1개의 block은 4KB이다.

### Hard Disk Structure

하드디스크의 구조는 아래와 같다. 여러 개의 sector가 보여 cluster를 이루고, sector 한 바퀴는 Track이다. Cylinder는 아래와 같이 여러 개의 원판에 존재하는 Track이다.

![image](https://user-images.githubusercontent.com/49841765/176985971-74fd6bba-6907-417a-b4a8-dd3f79b34fcb.png)

### Flie System Allocation

**Sequential Allocation**

정보가 연속적으로 저장되어 있는 구조이다. CD, DVD 등이 포함된다.

**Non-Sequential Allocation**

정보가 연속적으로 저장되어 있지 않고 흩어져 존재한다. index block, FAT (File Allocate Table) 등이 존재한다.

### Structure

데이터를 하나의 파일로 전체 디스크에 저장하는 것은 매우 비효율적인 방법이다. 그렇기 때문에 여러 개의 파일로 나눠 접근하는 방법이 필요하다. 이를 파일 시스템이라고 하는데, 운영체제마다 서로 다른 파일 시스템을 사용한다.

![image](https://user-images.githubusercontent.com/49841765/176985976-80c071a4-5d52-45c5-beb5-bbeb61b59497.png)

**partition**

하드디스크는 여러 개의 파티션으로 나누어져 있다.

**cylinder group**

cylinder group은 여러 개의 cylinder의 조합으로 구성된다.

**super block**

파일 시스템에 대한 정보를 포함하고 있는 block 이다.

ex) File System Type, data block 개수, inode 개수, inode table 시작 위치 등 ...

**boot block**

컴퓨터 부팅 시 필요한 부트스트랩 코드를 저장하고 있다. boot block은 어떤 종류의 파일 시스템이든 무조건 포함하고 있다.

`1sector` = 512 Bytes

`1cluster` = 8 sectors,

`1cluster` = 4KB

### Direct Block, Indirect Block

파일 크기가 block size인 4KB를 넘어갈 때, 연속되는 정보를 어떻게 참조해야할까. EXT 파일 시스템에서는 이를 Indirect Block을 통해 해결하고 있다.

![image](https://user-images.githubusercontent.com/49841765/176985984-e9813f07-24eb-4ea4-a5d1-8b0c3e2d1374.png)

block size는 4KB이고, 포인터 값은 4 Bytes를 갖는다. 즉, indirect block 하나 당 1024개의 포인터를 저장할 수 있다.

우선 파일의 크기가 4KB를 넘지 않는 파일은 Direct Block을 통해 접근할 수 있다. Direct Block은 데이터 블록의 포인터를 갖고, inode 하나 당 12개의 Direct Block 을 갖기 때문에, 48KB의 크기를 가진다.

파일의 크기가 4KB가 넘는 경우, 파일은 Indirect Block을 통해 접근 가능하다.

`Single Indirect Block`은 포인터 하나 당 데이터 블록 포인터 배열을 가진다. 그렇기 때문에 포인터 하나 당 1024 개의 데이터 블록에 접근할 수 있고, 총 1024 _ 1024 _ 4 = `4MB` 크기의 파일에 접근할 수 있다.

`Double Indirect Block`은 포인터 하나 당 Single Indirect Block의 포인터를 가진다. 총 1024 _ 1024 _ 1024 \* 4 = `4GB` 크기의 파일에 접근할 수 있다.

`Triple Indirect Block`은 포인터 하나 당 Double Indirect Block의 포인터를 가지고, `4TB` 크기의 파일에 접근할 수 있다.

### Directory

리눅스에서 directory는 파일로 간주된다. 즉, 고유한 inode number를 갖고, inode table entry를 갖는다.

![image](https://user-images.githubusercontent.com/49841765/176985987-ef8cb783-1086-4267-9bce-d6d1066e96bf.png)

directory는 데이터 블록에, 포함된 파일과 디렉토리의 i-node 정보를 담고 있다. 그렇기 때문에, directory 내부에서 접근하려고 할 때 해당 i-node 포인터로 데이터 블록에 접근이 가능하고, directory 일 경우 역시 접근 가능하다.
