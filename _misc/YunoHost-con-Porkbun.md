## Instalar [YunoHost](https://www.yunohost.org) con el [servicio de DNS dinámicas](https://github.com/porkbundomains/porkbun-dynamic-dns-python) de [Porkbun](https://www.porkbun.com)

Descarga la imagen de YunoHost desde [su servidor](https://yunohost.org/ru/install/hardware), la montas en nuestra tarjeta microSD siguiendo las instrucciones según tu sistema operativo. 

Si deseas conectar la máquina por WiFi en vez de cableado debes habilitarlo mediante la adición del archivo wpa_supplicant.conf (Si estás en Windows, habilita que se muestren las extensiones de archivo para comprobar que no acaba realmente en .txt) en el directorio raiz de tu tarjeta que contenga la siguiente información


```
country=es
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="NOMBREWIFI"
    psk="CONTRASEÑAWIFI"
}
```

Donde country es el código del país [según el standard ISO 3166-1 Alpha-2](https://www.iso.org/obp/ui/#search), [ssid el nombre de tu wifi, y psk su contraseña] (http://192.168.1.1/wifi5g.html)

Expulsas la unidad, la insertas en tu Raspberry y arrancas.

¡Ya ha comenzado la magia!
Una vez que la luz de verde de la Raspberry se apague, puedes comprobar en el [mapa de la red en el router](http://192.168.1.1/networkmap.html), la ip de la instalación de YunoHost, y si no, en una ventana de comando (cmd) ejecuta la orden
```
ping yunohost -4
```
la opción -4 fuerza el uso de ipv4 sobre ipv6 (parece que ahora ya viene por defecto en el comando ping)

Ahora que sabes la ip, o bien te diriges a la interfaz web de tu [yunohost.local](https://yunohost.local) o te conectas via ssh (consola de comandos) (con [Putty](www.putty.org), por ejemplo si estás en Windows) a la ip resultante del ping con el usuario root (root@XXX.XXX.XXX.XXX) o a root@yunohost.local en el puerto 22. Acepta la llave de seguridad del servidor ssh y conectate con la clave "yunohost".

### Opcional: Fijar la IP en el router
Si quieres no tener que estar buscando la ip de tu máquina puedes asignarle una IP fija dentro de tu red local. Para ello dirígete al router, opciones avanzadas, Advanced Seup, LAN, y añade a la lista de direcciones IP estáticas la dirección mac de tu dispositivo. (tu direcciçón MAC la puedes ver mediante el comando
```
ip link show wlan0
```
en la consola ssh
) No olvides darle a aplicar en la configuración del router y reiniciar la raspberry con el comando 
```
shutdown -r
```
Una vez reiniciada la Raspberry debería encontrarse en la IP que le acabas de asignar.


### Usar la API de [Porkbun](www.porkbun.com)
Como cliente de Porkbun se agradece la [API para configurar las DNS dinámicamente](https://github.com/porkbundomains/porkbun-dynamic-dns-python), aunque se echa de menos que esta no esté estandarizada para que pueda ser compatible con [lexicon](https://github.com/AnalogJ/lexicon), y autodetectada y configurada desde el propio YunoHost.

Las instrucciones son sencillas:
Primero deberíamos actualizar la lista de paquetes y las aplicaciones de nuestra máquina YunoHost mediante el comando en la consola ssh
```
sudo apt update
sudo apt full-upgrade
```
Si recibes unos comentarios de que han cambiado los repositorios y de que debes aceptarlo manualmente, es porque la version de debian que se usa ya no es la última, pero el full-upgrade lo soluciona.

Pasamos a la instalación del cliente de la API de Porkbun
ejecutas en la consola ssh de YunoHost

```
pip install requests
git clone https://github.com/porkbundomains/porkbun-dynamic-dns-python.git
cd porkbun-dynamic-dns-python/
sudo nano config.json.example
```

Ahora necesitas la clave API de Porkbun. Para ello vas a la página web de [Porkbun](www.porkbun.com), entras en tu cuenta, arriba en el menu "ACCOUNT" vas a "[API Access](https://porkbun.com/account/api)" y creas una nueva clave despues de darle un alias haciendo clic en el botón "Create API Key". Copia bien estos datos, ya que la clave "Secret Key" no se volverá a mostrar.

Añadimos los datos en la consola ssh copiándolos y pegándolos (clic derecho del ratón en la consola ssh aunque puede que te borre la linea en la que pegues).
Guardamos el documento con "Ctrl+X", "Y" para confirmar, y editas el nombre de archivo para quitar el .example y dejarlo como config.json, y confirmas con "Y".

Ahora, en el [gestor de dominios de Porkbun](https://porkbun.com/account/domainsSpeedy), debes habilitar la API ("API ACCESS") para el dominio que quieres usar en YunoHost.
Ejecutamos el script en la consola ssh del servidor YunoHost mediante

```
mv porkbun-dynamic-dns-python/ /usr/bin/
cd /usr/bin/porkbun-dynamic-dns-python
python porkbun-ddns.py config.json tudominio.com '*'
```

Ahora deberíamos programarlo para ejecutarlo al menos a cada reinicio. Para ello vamos a añadirlo a crontab. En la consola ssh de YunoHost escribe:
```
crontab -e
```

Elije tu editor preferido (el más fácil es nano) y añade al final del documento la línea:
```
@reboot python /usr/bin/porkbun-dynamic-dns-python/porkbun-ddns.py /usr/bin/porkbun-dynamic-dns-python/config.json tudominio.com '*'
0 */2 * * *  python /usr/bin/porkbun-dynamic-dns-python/porkbun-ddns.py /usr/bin/porkbun-dynamic-dns-python/config.json tudominio.com '*'
```
Guarda con la combinación "Ctrl+X" y acepta con "Y"
La primera línea ejecuta el script al reinicio y la segunda cada dos horas, por si acaso cambia de IP tu conexión.