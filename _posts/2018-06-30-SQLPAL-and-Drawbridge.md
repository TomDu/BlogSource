---
title: SQLPAL and Drawbridge
date: 2018-06-30 17:48:20
tags:
- SQLPAL
- Drawbridge
- WSL
- SQL Server on Linux
- SSIS on Linux
---

# Introduction

SQLPAL & Drawbridge is a virtualization technique which implements an virtualization layer that wraps unmodified Windows/Linux application into a container, so as to enable the app to run on Linux/Windows platforms, makes traditional applications easy to be cross-platform and well-isolated.

Drawbridge is a research prototype of a new form of virtualization for application sandboxing, raised and developed around 2011.

Then the SQL Server team launched a project named "Helsinki", wihch aimed to transplant SQL Server to Linux platform, making SQL Server a cross-platform database. They did it by integrating Drawbridge into SQL Server Operating System (SOS). SQLPAL is born.

<!-- more -->

> Making SQL Server run on Linux involves introducing what is known as a Platform Abstraction Layer (“PAL”) into SQL Server. This layer is used to align all operating system or platform specific code in one place and allow the rest of the codebase to stay operating system agnostic. Because of SQL Server’s long history on a single operating system, Windows, it never needed a PAL. In fact, the SQL Server database engine codebase has many references to libraries that are popular on Windows to provide various functionality. In bringing SQL Server to Linux, we set strict requirements for ourselves to bring the full functional, performance, and scale value of the SQL Server RDBMS to Linux. This includes the ability for an application that works great on SQL Server on Windows to work equally great against SQL Server on Linux. Given these requirements and the fact that the existing SQL Server OS dependencies would make it very hard to provide a highly capable version of SQL Server outside of Windows in reasonable time it was decided to marry parts of the Microsoft Research (MSR) project Drawbridge with SQL Server’s existing platform layer SQL Server Operating System (SOS) to create what we call the SQLPAL. The Drawbridge project provided an abstraction between the underlying operating system and the application for the purposes of secure containers and SOS provided robust memory management, thread scheduling, and IO services. Creating SQLPAL enabled the existing Windows dependencies to be used on Linux with the help of parts of the Drawbridge design focused on OS abstraction while leaving the key OS services to SOS. We are also changing the SQL Server database engine code to by-pass the Windows libraries and call directly into SQLPAL for resource intensive functionality.

Some products based on SQLPAL:
- SQL Server on Linux
- SQL Server Integration Services (SSIS) on Linux
- MySQL on Azure

On the other hand, Windows Subsystem for Linux was also implemented based on Drawbridge.

> As part of Project Drawbridge, the Windows kernel introduced the concept of Pico processes and Pico drivers. Pico processes are OS processes without the trappings of OS services associated with subystems like a Win32 Process Environment Block (PEB). Furthermore, for a Pico process, system calls and user mode exceptions are dispatched to a paired driver. Pico processes and drivers provide the foundation for the Windows Subsystem for Linux, which runs native unmodified Linux binaries by loading executable ELF binaries into a Pico process’s address space and executes them atop a Linux-compatible layer of syscalls.

So far we are able to run Ubuntu, Suse and Debian on latest Windows 10.

# Articals & blogs

- [Drawbridge](https://www.microsoft.com/en-us/research/project/drawbridge/)
- [SQL Server on Linux: How? Introduction](https://cloudblogs.microsoft.com/sqlserver/2016/12/16/sql-server-on-linux-how-introduction/)
- [Windows Subsystem for Linux Overview](https://blogs.msdn.microsoft.com/wsl/2016/04/22/windows-subsystem-for-linux-overview/)
- [Windows Subsystem for Linux Documentation](https://docs.microsoft.com/en-us/windows/wsl/about)
