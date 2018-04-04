# 1.介绍

&emsp;&emsp;安全增强型Linux（SELinux）是Linux内核中强制访问控制机制的实现，在检查标准自由访问控制后检查允许的操作。 SELinux可以根据定义的策略对Linux系统中的文件和进程以及他们的操作执行规则。

&emsp;&emsp;使用SELinux时，文件（包括目录和设备）被称为对象。进程（如运行命令或Mozilla Firefox应用程序的用户）称为主题。大多数操作系统使用自主访问控制（DAC）系统来控制主体如何与对象进行交互以及主体如何与对方进行交互。在使用DAC的操作系统上，用户控制他们拥有的文件（对象）的权限。例如，在Linux操作系统上，用户可以使他们的主目录具有全球可读性，使用户和进程（主体）可以访问潜在的敏感信息，而对这种不需要的操作没有进一步的保护。

&emsp;&emsp;仅依靠DAC机制对于强大的系统安全性基本上是不够的。 DAC访问决策仅基于用户身份和所有权，忽略其他与安全相关的信息，例如用户的角色，程序的功能和可信赖性以及数据的敏感性和完整性。每个用户通常对其文件具有完全的自由裁量权，因此很难执行系统范围的安全策略。此外，用户运行的每个程序都会继承授予用户的所有权限，并且可以自由更改对用户文件的访问权限，因此可以针对恶意软件提供最低限度的保护。许多系统服务和特权程序以超出其要求的粗粒度特权运行，因此可以利用这些程序中的任何一个的缺陷获得进一步的系统访问权限。^[1]^

&emsp;&emsp;以下是在不运行安全增强型Linux（SELinux）的Linux操作系统上使用的权限示例。这些示例中的权限和输出可能与您的系统略有不同。使用以下命令查看文件权限：

```shell
$ ls -l file1
-rwxrw-r-- 1 user1 group1 0 2009-08-30 11:03 file1
```

&emsp;&emsp;在这个例子中，前三个权限位`rwx`控制着Linux `user1`用户（在这种情况下，所有者）对`file1`的访问。接下来的三个权限位`rw-`控制Linux` group1`组必须对`file1`进行的访问。最后三个权限位`r—`控制其他人必须访问的`file1`，其中包括所有用户和进程。

&emsp;&emsp;安全增强型Linux（SELinux）为Linux内核添加了强制访问控制（MAC），并且在红帽企业版Linux中默认启用。通用MAC架构需要能够对系统中的所有进程和文件实施管理设置的安全策略，并根据包含各种安全相关信息的标签做出决定。正确实施后，系统可以充分保护自身，并通过防止篡改和绕过安全应用程序来为应用程序安全性提供关键支持。 MAC提供了强大的应用程序分离功能，可以安全执行不可信的应用程序。它限制与执行进程相关的特权的能力限制了利用应用程序和系统服务中的漏洞可能导致的潜在损害的范围。 MAC可以保护信息免受合法用户的限制授权，也可以防止合法用户无意中执行恶意应用。^[2]^

&emsp;&emsp;以下是运行SELinux的Linux操作系统上包含与进程，Linux用户和文件上使用的安全相关信息的标签示例。该信息被称为SELinux上下文，并使用以下命令查看：
```shell
$ ls -Z file1
-rwxrw -r-- user1 group1 unconfined_u：object_r：user_home_t：s0 file1
```
&emsp;&emsp;在这个例子中，SELinux提供了一个用户（`unconfined_u`），一个角色（`object_r`），一个类型（`user_home_t`）和一个级别（`s0`）。这些信息用于制定访问控制决策。通过DAC，访问仅受Linux用户和组ID的控制。记住在DAC规则之后检查SELinux策略规则是很重要的。如果DAC规则首先拒绝访问，则不会使用SELinux策略规则。

>注意
>
>在运行SELinux的Linux操作系统上，有Linux用户和SELinux用户。 SELinux用户是SELinux政策的一部分。 Linux用户被映射到SELinux用户。为了避免混淆，本指南使用Linux用户和SELinux用户区分两者。