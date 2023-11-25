# Transformation d'une OverTheBox Plus en Routeur avec OpenWrt

## Vue d'ensemble

Ces instructions permettent de convertir une OverTheBox (OTB) d'OVH, initialement conçue pour l'agrégation de liens, en un routeur utilisant OpenWrt. L'objectif est de réutiliser l'OTB lorsque le service OVH n'est plus en activité, en tirant parti de la flexibilité et des fonctionnalités étendues d'OpenWrt.

## Objectifs

- **Réutilisation de l'OverTheBox d'OVH** : Transformer efficacement l'OTB en fin de service.
- **Installation d'OpenWrt** : Installer le système d'exploitation OpenWrt sur l'OTB pour une personnalisation poussée et une gestion optimale du réseau.

## Prérequis

- Une OverTheBox Plus d'OVH inutilisée (assurez-vous qu'il s'agisse bien de ce modèle : [https://www.ovhtelecom.fr/overthebox/overTheBoxPlus.xml](https://www.ovhtelecom.fr/overthebox/overTheBoxPlus.xml)).
- Des connaissances de base en réseau et en systèmes d'exploitation Linux.
- Une clé USB équipée d'un système Linux (par exemple, une clé USB avec Ubuntu installé).

## Étapes

1. **Préparation de l'OverTheBox** : Vérifier que l'OTB est en état de marche et prête pour l'installation d'OpenWrt.

2. **Création d'un firmware OpenWrt** : Se rendre sur [https://firmware-selector.openwrt.org/?version=23.05.2&target=x86%2F64&id=generic](https://firmware-selector.openwrt.org/?version=23.05.2&target=x86%2F64&id=generic). (Version actuelle : 23.05.2)

3. **Ajout de paquets via "Personnaliser les paquets installés et/ou le script de démarrage"** :
Ajoutez les paquets suivants à ceux existants (sauf si, évidemment, vous savez ce que vous faites) :
```  
kmod-spi-ks8995 kmod-usb-net kmod-usb-net-asix luci-app-statistics collectd-mod-cpu collectd-mod-interface collectd-mod-load collectd-mod-memory collectd-mod-network collectd-mod-rrdtool collectd-mod-ethstat
```

4. **Ajout de scripts via "Script à exécuter au premier démarrage (uci-defaults)"** : 

```
# Configuration for VLAN 1 - Not used
uci add network switch_vlan
uci set network.@switch_vlan[-1].device='otbv2sw'
uci set network.@switch_vlan[-1].vlan='1'
uci set network.@switch_vlan[-1].ports='16 15t'

# Configuration for VLAN 2 - Lan
uci add network switch_vlan
uci set network.@switch_vlan[-1].device='otbv2sw'
uci set network.@switch_vlan[-1].vlan='2'
uci set network.@switch_vlan[-1].ports='1 2 3 4 5 6 7 8 9 10 11 12 17 18 15t'
uci set network.lan.device='eth0.2'
uci set network.lan.ipaddr='192.168.100.1'
uci set network.lan.netmask='255.255.255.0'

# Configuration for VLAN 3 - Wan Port 13
uci add network switch_vlan
uci set network.@switch_vlan[-1].device='otbv2sw'
uci set network.@switch_vlan[-1].vlan='3'
uci set network.@switch_vlan[-1].ports='13 15t'
uci set network.wan13=interface
uci set network.wan13.device='eth0.3'
uci set network.wan13.proto='dhcp'
uci del_list firewall.wan.network='wan13'
uci add_list firewall.wan.network='wan13'

# Configuration for VLAN 4 - Wan Port 14
uci add network switch_vlan
uci set network.@switch_vlan[-1].device='otbv2sw'
uci set network.@switch_vlan[-1].vlan='4'
uci set network.@switch_vlan[-1].ports='14 15t'
uci set network.wan14=interface
uci set network.wan14.device='eth0.4'
uci set network.wan14.proto='dhcp'
uci del_list firewall.wan.network='wan14'
uci add_list firewall.wan.network='wan14'
``` 

5. **Lancement de la création de l'image** : Cliquer sur "Demande de construction".
6. **Récupération de l'image souhaitée** : Personnellement, je préfère le '[COMBINED-EFI (EXT4)]'.
7. **Installation de l'image sur l'OTB** :  
   - Démarrer l'OTB via la clé USB Linux (appuyer sur la touche 'Suppr' au démarrage pour changer l'ordre de démarrage).
   - Sous Linux, supprimer les partitions de la MMC (via GParted ou un autre outil).
   - Flasher l'image sur la MMC (commande : `gunzip -c openwrt.img.gz | dd of=/dev/mmcblk0 bs=512`).

**Important : Après le flashage de la MMC, ne redémarrez pas l'appareil. Il sera nécessaire de commenter les deux lignes suivantes dans le fichier grub.cfg sur la MMC pour permettre le démarrage. Vous pouvez également redimensionner la partition /dev/mmcblk0p2 à l'aide d'un utilitaire tel que gparted pour lui attribuer la taille maximale de 7,1 Go.**

```
#serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1 --rtscts=off
#terminal_input console serial; terminal_output console serial
```

8. **Redémarrer l'otb** : Un reboot dans le terminal, l'OTB redémarre, l'accès se fera via 192.168.100.1, si vous ne l'avez pas changer dans la configuration précédente.  

## Contribution

Les retours, suggestions ou pull requests sont les bienvenus. Si vous avez des améliorations ou des compléments à proposer, n'hésitez pas à contribuer au projet.
