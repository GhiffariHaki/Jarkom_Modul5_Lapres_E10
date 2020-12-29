# Jarkom_Modul5_Lapres_E10
- Ardy Wahyu Setiawan 05111840000050
- Mochamad Haikal Ghiffari 05111840000095

## Topologi
```
# Switch
uml_switch -unix switch0 > /dev/null < /dev/null &
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &
uml_switch -unix switch4 > /dev/null < /dev/null &
uml_switch -unix switch5 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.70.45 eth1=daemon,,,switch0 eth2=daemon,,,switch3 mem=96M &
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch3 eth1=daemon,,,switch2 eth2=daemon,,,switch5 mem=96M &
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch0 eth1=daemon,,,switch1 eth2=daemon,,,switch4 mem=96M &

# Server
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch1 mem=128M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 mem=128M &
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &

# Klien 1
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch4 mem=96M &
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch5 mem=96M &
```

## VLSM

A. Tree

![image](https://user-images.githubusercontent.com/57068224/103280480-199cdd00-4a03-11eb-8ced-ef9128bfce39.png)

B. Pembagian IP

![image](https://user-images.githubusercontent.com/57068224/103280502-27526280-4a03-11eb-81e6-115d23062df1.png)

## Routing
```
SURABAYA
A2 A3 A5 A6
route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.0.2
route add -net 192.168.2.0 netmask 255.255.255.0 gw 192.168.0.6
route add -net 192.168.0.8 netmask 255.255.255.248 gw 192.168.0.2
route add -net 10.151.71.88 netmask 255.255.255.248 gw 192.168.0.6

ip route add 192.168.1.0/24 via 192.168.0.2
ip route add 192.168.2.0/24 via 192.168.0.6
ip route add 192.168.0.8/29 via 192.168.0.2
ip route add 10.151.71.88/29 via 192.168.0.6

KEDIRI
route add -net 0.0.0.0 netmask 0.0.0.0 gw 192.168.0.1
ip route add 0.0.0.0 via 192.168.0.1

BATU
route add -net 0.0.0.0 netmask 0.0.0.0 gw 192.168.0.5
ip route add 0.0.0.0 via 192.168.0.5
```
## DHCP

- Server di Mojokerto

![image](https://user-images.githubusercontent.com/57068224/103280604-6aacd100-4a03-11eb-8da5-ce0793fc6902.png)
![image](https://user-images.githubusercontent.com/57068224/103280609-6d0f2b00-4a03-11eb-993d-a8c8b00550c2.png)
![image](https://user-images.githubusercontent.com/57068224/103280614-6ed8ee80-4a03-11eb-939d-c1fdf5d9e3c9.png)

- Relay di Surabaya, Batu, Kediri

![image](https://user-images.githubusercontent.com/57068224/103280621-71d3df00-4a03-11eb-9450-c9a2e9b46a56.png)

## Soal Shift
1. Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi
SURABAYA menggunakan iptables, namun Bibah tidak ingin kalian menggunakan
MASQUERADE.
```
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.70.46 #surabaya
```
![image](https://user-images.githubusercontent.com/57068224/103280834-f45c9e80-4a03-11eb-874f-c6e4fc3545c4.png)

2. Kalian diminta untuk mendrop semua akses SSH dari luar Topologi (UML) Kalian pada server
yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan.
```
iptables -A FORWARD -d 10.151.71.88/29 -i eth0 -p tcp --dport 22 -j DROP #surabaya
```
![image](https://user-images.githubusercontent.com/57068224/103280842-f6bef880-4a03-11eb-9361-05d4e39286bd.png)
3. Karena tim kalian maksimal terdiri dari 3 orang, Bibah meminta kalian untuk membatasi DHCP
dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari
mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP # mojo malang
```
![image](https://user-images.githubusercontent.com/57068224/103280902-17874e00-4a04-11eb-88c4-1ed996adb842.png)

4. Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin
sampai Jumat.
```
iptables -A INPUT -s 192.168.2.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -s 192.168.2.0/24 -j REJECT # malang test sidoarjo

```
![image](https://user-images.githubusercontent.com/57068224/103280908-19e9a800-4a04-11eb-8dbb-6b26b07b19fd.png)
![image](https://user-images.githubusercontent.com/57068224/103280914-1c4c0200-4a04-11eb-80b1-483a1b9c4be5.png)

5. Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap
harinya.
```
iptables -A INPUT -s 192.168.1.0/24 -m time --timestart 07:00 --timestop 17:00 -j REJECT # malang tes gresik
```
![image](https://user-images.githubusercontent.com/57068224/103280924-240ba680-4a04-11eb-9b21-c1d52bd80207.png)
![image](https://user-images.githubusercontent.com/57068224/103280930-279f2d80-4a04-11eb-97ec-2b8d5c024d61.png)

6. Bibah ingin SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan
secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80.
```
iptables -A PREROUTING -t nat -p tcp -d 10.151.71.90 --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.0.11:80 #Probolinggo
iptables -A PREROUTING -t nat -p tcp -d 10.151.71.90 --dport 80 -j DNAT --to-destination 192.168.0.10:80 #Madiun

iptables -t nat -A POSTROUTING -p tcp -d 192.168.0.11 --dport 80 -j SNAT --to-source 10.151.71.90:80 #Probolinggo
iptables -t nat -A POSTROUTING -p tcp -d 192.168.0.10 --dport 80 -j SNAT --to-source 10.151.71.90:80 #Madiun
```

7. Bibah ingin agar semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap
UML yang memiliki aturan drop.
```
iptables -N LOGGING
iptables -A INPUT -j LOGGING
iptables -A OUTPUT -j LOGGING
iptables -A LOGGING -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP #sby malang mojo
```
![image](https://user-images.githubusercontent.com/57068224/103280991-4ac9dd00-4a04-11eb-8d31-4416f022b310.png)
