# Windows下mklink使用 #

## mklink命令 ##

mklink是windows系统下创建符号链接和硬链接的命令工具，它是一个很好的解决文件系统问题的工具。使用它需要管理员权限。

首先，先来介绍下mklink这个命令：

> mklink
> 创建符号链接。
> mklink [[/D] | [/H] | [/J]] Link Target
> /D 创建目录符号链接。默认为文件符号链接。
> /H 创建硬链接而非符号链接。
> /J 创建目录联接。
> Link 指定新的符号链接名称。
> Target 指定新链接引用的路径(相对或绝对)。

> 注意：因为powershell不支持mklink命令，所以要在前面加cmd /c表示用cmd来运行该命令，路径注意引号，可以是相对路径也可以是绝对路径。

## 符号链接（Symbolic link） ##

对文件创建符号链接


	mklink "link" "target"

对文件夹创建符号链接

	mklink /D "link" "target"

符号链接连接远程的路径

	mklink /D "D:\link" "\\123.123.0.1\D$\target"

特点

- 创建链接后的图标和快捷方式很像, 都有一个箭头的标志
- 在系统中不占用空间
- 在文件系统中不是一个单独的文件
- 如果源文件被删除了，链接就没用了
- 移除源文件不会影响符号链接
- 移除链接文件也不会影响源文件
- win10_x64_build10565上测试不可以右键修改图标和设置管理员运行
- 文件大小为0字节和不占用空间
- 文件属性的创建时间和修改时间都是软链接创建和修改时的时间
- 文件类型是.SYMLINK
- 可以在cmd下运行软链接(假如链接的是程序, 且运行命令是XXX即可)(win10_x64_build10565上测试通过)

符号链接是在文件系统上实现的链接，对操作系统上大多数软件来说是透明的，也就是说，当软件访问符号链接时，其实际上是在访问该符号链接所指向的文件夹。

创建的符号链接显示的类型是文件夹，实际上相当于是指向其他盘真实的文件夹路径的快捷方式，符号链接本身不占空间。路径映射的过程对程序来说是透明的，程序对这个符号链接的操作实际上是对文件夹的操作，因此程序可以正常运行，所以这和普通的创建快捷方式是不一样的操作。此外，符号链接和目录联接是有快捷方式的那个箭头的。

> 注意：软链接的创建需要管理员权限，确保cmd是管理员模式。对于文件夹的软链接创建，一定要加上”/D”。通过相对路径创建的软链接在移动后无法使用，绝对路径创建的移动后不影响使用。符号链接可以直接右键删除，或通过rmdir命令删除，不会影响原文件，但del命令则会把目标文件删除。

## 硬链接（Hard link） ##

对文件创建硬链接

	mklink /H "link" "target"

- 硬链接只能用于文件，不能对文件夹创建硬链接，不然会提示“拒绝访问”
- 在系统中占用的空间与源文件相同，但在系统中引用的是相同的对象（不是拷贝）
- 图标和创建快捷方式的图标不同(没有快捷方式的小箭头)
- 移除源文件不会影响硬链接
- 移除硬链接不会影响源文件
- 如果源文件被删除，它的内容依然通过硬链接存在
- 硬链接文件的任何更改都会影响到源文件
- 文件大小, 占用空间, 创建和修改时间跟原原文件一样
- 可以在cmd下运行硬连接(假如链接的是程序)


通过上述命令就可以创建从“link”路径到“target”路径的硬链接，例如：在D盘根目录下新建文本“A.txt”，然后输入命令如下即可创建到”A.txt”的硬链接”B.txt”

在文件资源管理器上看，“B.txt”与“A.txt”占用同样大小的空间，其实这个数据并不用去理会，硬链接相当于给文件的数据多创建了一个“入口”，“A.txt”,“B.txt”指向的是硬盘中的同一块区域，因此这两个文件的内容是完全一样的，编辑任何一个文件都会影响到另一文件，当删除其中一个文件，只是删除这个文件其中一个“入口”，要两个文件都删除，文件系统才会标志这块硬盘区域上的文件被删除。

## 目录联接(创建软链接首选) ##

对文件夹创建目录联接

	mklink /J "link" "target"

“目录联接”只能应用于文件夹，不可用于文件。根据网上能找到的资料显示，对文件夹创建的“目录联接”与“符号链接”并没有区别，一样可以实现软件数据的迁移。不过貌似这两者对剪切操作有不一样的表现。

例如，我现在在D盘创建“文件夹A”,在“文件夹A”里新建A.txt，然后在D盘根目录创建一个“目录联接B”指向这个“文件夹A”，通过这个“目录联接B”，我可以访问到A.txt，接着我对“目录联接B”进行剪切操作，剪切到C盘，发现“文件夹A”和“目录联接B”还是在D盘，但是打开却发现A.txt不见了，被剪切到了C盘的“文件夹B”中，也就是说对“目录联接”的剪切操作会影响原来的文件。

对于这其中的机制，很神奇。。。感觉“目录联接”跟“符号链接”有点像，给文件夹里的内容提供一个“入口”即所谓的“联接点”，剪切操作时会通过这个“联接点”把内容剪切出来，原来的目录和“联接点”虽然没有变化，但里面的内容被剪切出来了。

而“符号链接”的剪切操作仅仅是对这个“符号链接”的剪切，并不会透过这个“符号链接”把其内容剪切掉。

## mklink 硬链接和符号链接的区别 ##

硬链接只能用于文件，不能用于文件夹，而且硬链接和目标文件必须在同一个分区或者卷中。硬链接的目的是为了给文件创建多个目录路径(多个入口)，而不像符号链接是为了指向某个已有的文件。

假设要给Target. txt文件创建一个硬链接，系统下载可以执行以下命令：

	mklink/H Link.txt Target.txt

和符号链接一样，硬链接中所做的任何修改，都会自动应用到目标文件上。但是硬链接具有以下一些不同的地方。

(1)硬链接必须引用同一个分区或者卷中的文件，而符号链接可以指向不同分区或者共享文件夹上的文件或者文件夹。

(2) 硬链接只能引用文件，而符号链接可以引用文件或者文件夹。

(3)Windows会自动维护硬链接，即使把硬链接复制到其他文件夹，硬链接和目标都可以继续访问。

(4)删除目标文件，硬链接可以继续保留。只有把目标文件和所有的硬链接都删除，才能把该文件彻底删除。

(5)如果win7把符号链接的目标文件删除，然后用一个同名文件替换，则符号链接会指向新的目标文件；而把硬链接的目标文件删除’再用同名文件替换，则硬链接还是会继续引用原始文件。

(6)也就是说，硬链接和目标文件的地位相等。事实上，原始的目标文件本身也相当于硬链接，新建硬链接，只是相当于增加一个目录路後而已。

(7)硬链接看上去和真的文件一模一样(实际上就是真实的文件)，不像符号链接那样有一个快捷方式的小箭头，但是硬链接并不会增加磁盘空间的占用。

(8)对硬链接进行NTFS权限的修改，会同时影响到目标文件(因为两者等价)，而符号链接和目标文件可以设置不同的NTFS权限。

> 综上，我们可以将硬链接理解成C语言中的指针、空间中的传送门，文件随着硬链接的创建，等于它有一个“固定”地址，但它对外沟通的通道有多个，只要通道还存在，该文件就不会消失。

## mklink /D和/J的区别 ##

目录符号链接和目录联接的区别在于：

目录联接在创建时会自动引用目标目录的绝对路径，而符号链接允许相对路径的引用。

如分别用 mklink /D dira tdir 和 mklink /J dirb tdir 创建 dira、dirb 对相对目录的 tdir 的符号链接和目录联接，之后将 dira、dirb 移动到其它目录下，则访问 dira 时会提示“位置不可用”，访问 dirb 时仍然正常指向 tdir；

且win10_x64_build10565的cmd下dir命令查看会发现, dira符号链接链接到的是相对路径下的tdir文件(不管是否存在tdir文件), 且文件类型是symlink, dirb目录联接则链接到绝对(全)路径下的tdir文件, 且文件类型是junction(可能是系统自动把相对路径转换为全路径)

而分别用 mklink /D dira c:\demo\tdir 和 mklink /J dirb c:\demo\tdir 创建 c:\demo\tdir 的符号链接和目录联接，再将这两个目录链接移动到其它目录下，则 dira 和 dirb 均可正常指向 c:\demo\tdir；

由此可见当创建目录链接时对目标目录使用绝对路径，D 和 J 两个参数实现的目录链接效果是一样的；

英文原文：

	MKLINK /D | /H | /J Link Target

/D Creates a directory symbolic link. Default is a file symbolic link. /H Creates a hard link instead of a symbolic link. /J Creates a Directory Junction.

/D creates a symbolic link, or a soft link.This essentially acts like a shortcut to a folder in prior versions of Windows, except you don’t have to use an actual shortcut.

/H creates a hard link, which points directly to the file.This option can’t be used for folders directly for some reason, you’ll have to use the next option.

/J creates a “Directory Junction”A Directory Junction is actually just a hard link to a directory. This is a feature that existed prior to Vista as well. If you are trying to symlink to a directory using a hard link, then you should use this option.

Understanding Hard vs Soft Links================================Hard Link

A hard link directly points to the file, and acts to the operating system as if it is the file itself. You’ll want to use this option the majority of the time if you are trying to fake an application’s directory.

Soft Link

A soft link is essentially a shortcut to a file or folder – if you are using Windows explorer, you’ll be redirected to the directory if you double-click on a shortcut, it won’t pretend its part of the filesystem. You can still directly reference or open a file with the symlinked path, and it mostly works.

转载请注明来源，欢迎对文章中的引用来源进行考证，欢迎指出任何有错误或不够清晰的表达。可以邮件至 qasdwasd@qq.com

————————————————

版权声明：本文为CSDN博主「星幕-云影」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/weixin_45385155/article/details/105559446