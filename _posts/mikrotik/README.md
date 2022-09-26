---
layout: post
title: "MikroTik 7 VPN WireGuard Server & Client"
date: 2022-09-26 14:23:00 +0700
categories: [mikrotik, wireguard, android, linux, vpn]
tags: mikrotik
author: toni
permalink: /mikrotik-wireguard-windows-android/
---

* TOC
{:.toc}

# Setup MikroTik Server

source: [https://citraweb.com/artikel/407/](https://citraweb.com/artikel/407/)

Login Ke mikrotik server yang punya ip public static & bisa diakses, bisa dg chr pada vps seperti digitalocean, aws, azure dll. atau bisa menggunakan ubuntu/debian sbg wireguard server pada vps tersebut.

dokumentasi ini ditulis pada mikrotik versi 7.5 dan winbox versi 3.31.

tambahkan interface wireguard pada menu WireGuard di winbox

![1-server-mikrotik](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/1-server-mikrotik.png)

klik tanda + "add" untuk menambahkan interface

![2-interface-server](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/2-interface-server.png)

pada `interface name` secara default dimulai dari 1, misal `wireguard1`. `listen port` bisa dibiarkan default `12321` atau disini kita rubah menjadi `52252`

    nb: pastikan port ini dibuka pada firewall:
    ip/firewall/filter
    add chain=input in-interface=ether1 protocol=tcp dst-port=52252 action=accept
    add chain=input in-interface=ether1 protocol=udp dst-port=52252 action=accept

![3-interface-wireguard](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/3-interface-wireguard.png)

setelah klik `apply`, akan di generate otomatis `private key` dan `public key`, copy `public key` ke notepad.

lalu tambahkan `ip address` pada interface `wireguard1`, misal address "10.10.10.1" dan network "10.10.10.2" untuk network point-to-point 1:1 misal untuk site-to-site network/jaringan pusat-ke-cabang. dg peer nantinya hanya ada 1 untuk ip ini.

atau gunakan address "10.10.10.1/24" jika nantinya akan digunakan oleh beberapa peer.

![4-add-ip-address](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/4-add-ip-address.png)

# Setup MikroTik Client

setup wireguard pada mikrotik sbg client.

pertama buat interface `wireguard1` pada menu wireguard->interface, jika belum ada.

![5-mikrotik-client](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/5-mikrotik-client.png)

klik `apply`, copy `public key` ke notepad, klik `ok` utk menutup window ini, lalu berpindah ke tab `peers`

klik "+" atau add, pilih interface `wireguard1` (sesuaikan nomor interface). pastekan `public key` server disini,
masukkan juga ip public server pada `endpoint` dan port server "52252" tadi pada `endpoint port`, pada `allowed address` bisa diinput network ip `10.10.10.0/24` dahulu, kecuali nantinya kita akan gunakan utk me-routing ip lokal tertentu pada skema site-to-site networking.

![6-interface-client](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/6-interface-client.png)

tambahkan ip pada interface `wireguard1` pada menu "ip->address", misal address 10.10.10.2 dan network 10.10.10.1 sbg pasangan ip pada sisi server tadi.
atau gunakan address "10.10.10.2/24" untuk network yg lebih luas/peers yg lebih dari satu.

![7-ip-address-client](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/7-ip-address-client.png)

lakukan ping dari sisi server ke client, pastikan ip client sudah bisa reply

![8-test-ping-ke-client](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/8-test-ping-ke-client.png)

lakukan ping dari sisi client ke server, pastikan ip server sudah reply

![9-test-ping-ke-server](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/9-test-ping-ke-server.png)

    catatan: ada beberapa skema yg bisa dibuat dengan mengatur ip address pada interface wireguard,
    misal kita ingin hanya ada 1 peer untuk 1 interface, kita bisa setting ip address misal,
    pada server: address 10.10.10.3 network 10.10.10.4 interface wireguard2 (sesuaikan nama interface)
    pada client: address 10.10.10.4 network 10.10.10.3 interface wireguard1 (sesuaikan nama interface)
    dst, dg begini 1 port pada ip public hanya dapat digunakan oleh 1 peer, atau 1 koneksi saja


    catatan 2: pada skema lain, misal dg ip address 10.10.10.0/24 kita bisa membatasi akses peer hanya ke
    ip address server saja, dengan mengatur "allowed address 10.10.10.1" (ip wireguard1 server),
    sehingga ip peer lain tidak bisa saling akses 

# Setup Android Client

![18-mikrotik-peer-android](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/18-mikrotik-peer-android.png)

# Setup Windows 10 Client

source: [https://www.smarthomebeginner.com/wireguard-windows-setup/](https://www.smarthomebeginner.com/wireguard-windows-setup.png)

download installer wireguard untuk windows [disitus resmi wireguard](https://www.wireguard.com/install/)

![10-windows-download](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/10-windows-download.png)

setelah download selesai, install exe/msi sebagai mana mestinya :)

buka aplikasi wireguard, sampai tampilan berikut, lalu klik "add tunnel -> add empty tunnel" atau tekan "Ctrl+N"

![11-windows-gui](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/11-windows-gui.png)

vpn name: core-router
copy public key ke notepad, simpan nanti kita paste sbg peers di server.

berikut contoh konfigurasi peer:

![12-windows-client-config](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/12-windows-client-config.png)

```
[Interface]
PrivateKey = [digenerate otomatis, tidak perlu dirubah]
Address = 10.10.10.3/24 [ip address kita]

[Peer]
Endpoint = [ip public server]:52252
AllowedIPs = 10.10.10.1/24
PublicKey = [public key server]
```

kita bisa tambahkan peer komputer lain di jaringan kita, yaitu pc tanpa ip public, atau pc dg ip lokal.
misal membuat pc kita jadi "server/listen" peer

```
[Interface]
ListenPort = 52255
```

lalu pada pc lain, masukkan ip pc kita sbg peer/server, misal ip kita 192.168.100.2

```
[Peer]
Endpoint = 192.168.100.2:52255
AllowedIPs = 10.10.10.1/24
PublicKey = [public key pc kita]
```

tambahkan public key kita di server mikrotik

![13-mikrotik-peer-windows](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/13-mikrotik-peer-windows.png)

aktifkan vpn wireguard kita, disini sbg contoh kita tambahkan "AllowedIPs" berupa network yang akan kita akses pada sisi server

![14-windows-activate](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/14-windows-activate.png)

![15-windows-connected](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/15-windows-connected.png)

Coba lakukan ping ke ip di network server melalui vpn

![16-windows-test-ping](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/16-windows-test-ping.png)

cek ip interface wireguard kita

![17-windows-interface](https://raw.githubusercontent.com/aliffathon/is/gh-pages/_posts/mikrotik/img/17-windows-interface.png)

# Setup Linux Server Client