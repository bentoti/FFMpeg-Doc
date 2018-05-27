# CENTOS 7 安装NVIDIA驱动


* 下载NVIDIA驱动:

    官网下载地址：[http://www.geforce.cn/drivers](http://www.geforce.cn/drivers)。根据自己的显卡型号，选择相应的显，进行下载。


* 安装EPEL源

    ```
    [root@localhost ~]# yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    ```

* 安装编译环境

    ```
    [root@localhost ~]# yum -y install gcc kernel-devel kernel-headers dkms
    ```

    安装完确认kernel和kernel-devel、kernel-headers版本一致。

    ```
    [root@localhost ~]# uname -r

    3.10.0-862.3.2.el7.x86_64
    ```

    ```
    [root@localhost ~]# rpm -qa | grep kernel

    kernel-tools-libs-3.10.0-862.3.2.el7.x86_64

    kernel-devel-3.10.0-862.3.2.el7.x86_64

    kernel-3.10.0-862.3.2.el7.x86_64

    kernel-tools-3.10.0-862.3.2.el7.x86_64

    kernel-headers-3.10.0-862.3.2.el7.x86_64

    kernel-debug-devel-3.10.0-862.3.2.el7.x86_64
    ```

    若版本不一致，请重新安装或升级使其版本一致。

* 禁用nouveau

    修改/etc/modprobe.d/blacklist.conf 文件，添加blacklist nouveau，注释掉blacklist nvidiafb。

    如果blacklisst.conf不存在，执行下面的命令

    ```
    [root@localhost ~]# echo -e "blacklist nouveau\noptions nouveau modeset=0" > /etc/modprobe.d/blacklist.conf
    ```

    若安装了图形界面，修改为以字符界面启动，然后重启

    ```
    [root@localhost ~]# ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target

    [root@localhost ~]# reboot
    ```

    开机后若下面的命令没有输出，nouveau禁用成功

    ```
    [root@localhost ~]# lsmod | grep nouveau
    ```

* 重新建立initramfs image文件

    ```
    [root@localhost ~]# mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak

    [root@localhost ~]# dracut /boot/initramfs-$(uname -r).img $(uname -r)
    ```

* 执行安装脚本

    ```
    [root@localhost ~]# ./NVIDIA-Linux-x86_64-375.39.run --kernel-source-path=/usr/src/kernels/3.10.0-514.el7.x86_64 -k $(uname -r) --dkms -s
    ```

    --kernel-source-path参数根据自己的路径修改
    -s 去掉-s以交互模式安装

* 安装完成

    ```
    [root@localhost ~]# nvidia-smi

    Sun May 27 21:35:35 2018       

    +-----------------------------------------------------------------------------+

    | NVIDIA-SMI 390.59                 Driver Version: 390.59                    |

    |-------------------------------+----------------------+----------------------+

    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |

    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |

    |===============================+======================+======================|

    |   0  GeForce GTX 980M    Off  | 00000000:01:00.0 Off |                  N/A |

    | N/A   55C    P0    26W /  N/A |      0MiB /  4046MiB |      0%      Default |

    +-------------------------------+----------------------+----------------------+

    +-----------------------------------------------------------------------------+

    | Processes:                                                       GPU Memory |

    |  GPU       PID   Type   Process name                             Usage      |

    |=============================================================================|

    |  No running processes found                                                 |

    +-----------------------------------------------------------------------------+
    ```


* 问题总结

    ```
    ERROR: Unable to load the kernel module 'nvidia.ko'.
    ```
    nouveau模块未禁用

    ```
    Unable to load the 'nvidia-drm' module
    ```

    没有安装dkms模块

    ```
    ERROR: Failed to run '/sbin/dkms build -m nvidia -v 375.66 -k 3.10.0-514.el7.x86_64': Error! echo
    Your kernel headers for kernel 3.10.0-514.el7.x86_64 cannot be found at
    /lib/modiles/3.10.0-514.el7.x86_64/build or
    /lib/modiles/3.10.0-514.el7.x86_64/source.

    ERROR: Failed to install the kernel module through DKMS. No kernel module
    was installed; please try installing again without DKMS, or check the DKMS logs for more information.

    ERROR: Installation has failed. Please see the file
    '/var/log/nvidia-installer.log' for details. You may find
    suggestions on fixing installation problems in the README available
    on the Linux driver download page at www.nvidia.com.
    ```

    内核等版本不一致
