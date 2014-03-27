---
layout: post
title: "raspberry pi wireless"
date: 2014-03-25 22:36
comments: true
categories: raspberrypi memo
tags: pi
---

前几天入手了Raspberry Pi，由于住的地方是无线路由，路由器又不在我的房间不方便接跟网线上去，想在家里玩的话就得把无线链接先配置好。

Raspberry Pi mode B国产红色的那款，操作系统raspbian wheezy 3.10，无线网卡用的edup的一款。

第一次进去当然还是要用网线才行，sshd是默认启动的，可以ssh进去，或者装个vnc server实现远程桌面，配置好了后就可以用自己的笔记本当显示器来用了。

使用raspbian自带的<u>wap_suppliant</u><sup>[1](#1)</sup> 配置无线网络
<!-- more -->
	
	$ cat /etc/network/interfaces
	auto lo

	iface lo inet loopback
	iface eth0 inet dhcp

	allow-hotplug wlan0
	iface wlan0 inet manual
	wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
	iface default inet dhcp	
	
	$ cat /etc/wpa_supplicant/wpa_supplicant.conf
	ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
	update_config=1
	network={
        ssid="office-wifi"
        scan_ssid=1
		key_mgmt=WPA-PSK
        psk=""
        id_str="office-wifi-password"
        priority=5
	}

	network={
        ssid="home-wifi"
        scan_ssid=1
		key_mgmt=WPA-PSK
        psk="home-wifi-password"
        id_str="home"
        priority=5
	}

配置好ssid和psk，插上电源启动raspberry就会自动连接到合适的无线网络了。进路由器管理界面可以看到连接设备里有个raspberrypi可以看到获得的ip。

<a id="1">[1] [wpa_suppliant](http://hostap.epitest.fi/wpa_supplicant/)
