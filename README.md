# ATAQUE DHCP STARVATION HECHO POR ÓSCAR GIL

  ![dhcp-750x445](https://github.com/user-attachments/assets/8bda0d8d-64fd-4bd5-a073-7c2ae86937cc)


## EXPLICACIÓN 

El ataque de DHCP Starvation es un tipo de ataque de denegación de servicio (DoS) que tiene como objetivo agotar el pool de direcciones IP disponibles de un servidor DHCP.

Este ataque se lleva a cabo cuando un atacante envía de forma masiva solicitudes DHCP (DHCP Discover) utilizando direcciones MAC falsas o aleatorias. El servidor DHCP, al recibir estas solicitudes, asigna una dirección IP a cada una de ellas, creyendo que se trata de nuevos dispositivos legítimos que intentan conectarse a la red.

Como consecuencia, el servidor DHCP termina consumiendo todas las direcciones IP disponibles en su rango de asignación. Una vez agotado el pool, los dispositivos legítimos ya no pueden obtener una dirección IP, impidiéndoles acceder correctamente a la red.

## ESQUEMA

Para realizar este laboratorio se han utilizado las siguientes maquinas virtuales y herramientas:

- **Máquina Atacante**: Ubuntu Cliente en red interna, utilizando la herramienta Yersinia en modo gráfico.
  
  <img width="580" height="503" alt="3" src="https://github.com/user-attachments/assets/0fac0b83-5140-4882-8542-698ec19f8d27" />
- **Servidor DHCP**: Ubuntu Server configurado con un pool de direcciones IP en red interna.
  
  <img width="623" height="163" alt="10" src="https://github.com/user-attachments/assets/6b310dd9-d443-4402-af91-cf5db90331d0" />
- **Máquina Víctima**: Windows utilizado para comprobar que no se obtiene dirección IP durante el ataque.

- **Yersinia**: Framework de pentesting especializado en la explotación de vulnerabilidades en protocolos de red como DHCP, STP...

## PRÁCTICA

Para empezar nos aseguramos de actualizar el sistema e instalar el DHCP en el servidor DHCP.

  <img width="801" height="298" alt="2" src="https://github.com/user-attachments/assets/79b76812-bf4d-4465-9b45-1311f5a00368" />

```bash
sudo apt update
sudo apt install isc-dhcp-server -y
```

Ahora editamos el archivo de configuración del DHCP que se encuentra en /etc/dhcp/dhcpd.conf.

  <img width="811" height="605" alt="4" src="https://github.com/user-attachments/assets/54e4cd18-29b8-4f62-812c-2dd1f74c1951" />

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Despues nos dirigimos a la interfaz y la indicamos

  <img width="669" height="332" alt="12" src="https://github.com/user-attachments/assets/086bfb93-386e-497f-91cd-83544ad4a2c3" />

```bash
sudo nano /etc/default/isc-dhcp-server
```

Cuando terminamos de configurar el servidor DHCP, reiniciamos el servicio DHCP para que se apliquen los cambios.

```bash
sudo systemctl restart isc-dhcp-server
```

Continuamos con la maquina atacante. Para empezar instalamos nuestra herramienta de ataque en este caso Yersinia en modo gráfico.

  <img width="726" height="382" alt="5" src="https://github.com/user-attachments/assets/ba3da857-9f39-4506-986e-1bedaaf79a69" />

```bash
sudo apt update
sudo apt install yersinia
```

Una vez instalados, lo iniciamos.

  <img width="1149" height="144" alt="16" src="https://github.com/user-attachments/assets/b72f6c19-bb48-4cb5-8023-96759b12d00e" />

```bash
sudo yersinia -G
```

Importante, antes eliminar la IP de la victima para que funcione correctamente.

  <img width="566" height="186" alt="18" src="https://github.com/user-attachments/assets/8f587029-72a9-4f39-8d21-5d9d93d4b300" />

```bash
Ipconfig /release
```

Y ademas, se desactiva el adaptador de red para forzar a borrar la IP.

  <img width="359" height="448" alt="19" src="https://github.com/user-attachments/assets/bbd59622-af44-466b-9c4d-537b99de7aa7" />

Seguimos con el ataque, entramos en Launch Attack y nos vamos a DHCP Discover.

  <img width="1273" height="580" alt="24" src="https://github.com/user-attachments/assets/27e44151-c2d2-4abf-bd82-8869e4938e59" />

Una vez funcionando el ataque, comprobamos desde el Ubuntu Server como está dejando sin IPs al DHCP.

  <img width="804" height="502" alt="6" src="https://github.com/user-attachments/assets/24fa3224-9eee-4171-8fff-faf1ba9096a6" />

Volvemos a activar el adaptador de red en el Windows. 

 <img width="365" height="457" alt="20" src="https://github.com/user-attachments/assets/1d1732e3-a409-48ca-bd86-32be7dc2cfcb" />

Y pedimos que nos asignen una IP del DHCP Server pero como no quedan leases nunca asignará.

  <img width="642" height="175" alt="14" src="https://github.com/user-attachments/assets/4c08498d-c083-4f3d-b0cd-182b3f3f62b8" />

```bash
Ipconfig /release
```

Para acabar con el ataque, desactivamos el adaptador de red y pausamos el ataque.

  <img width="791" height="422" alt="22" src="https://github.com/user-attachments/assets/4608f5da-51f2-4ac5-a6fd-7dc6e4f6cc44" />

Cuando se pare, activamos el adaptador de red y pedimos una IP al servidor DHCP.

  <img width="558" height="161" alt="21" src="https://github.com/user-attachments/assets/35ba76e4-bdf9-47e2-9089-f0d9a28410be" />

```bash
Ipconfig /renew
```

## MITIGACIÓN DEL ATAQUE DHCP STARVATION

Para mitigar el ataque de DHCP Starvation vamos a tocar la configuración del servidor DHCP.

Nos vamos a /etc/dhcp/dhcpd.conf y añadimos esta configuración.

  <img width="802" height="605" alt="23" src="https://github.com/user-attachments/assets/ca311bed-237a-4fa4-9081-1a1924ce1198" />

```bash
deny unknown-clients;
host cliente_victima {
   hardware ethernet 08:00:27:8F:50:AE;
   fixed-address 192.168.1.125;
}
```

Cuando volvemos a atacar con Yersinia en modo gráfico no permitirá las entradas de solicitudes DHCP de clientes no conocidos.

Haciendo una liberación y renovacion de IP comprobamos que recibe IPs durante el ataque.

  <img width="555" height="230" alt="25" src="https://github.com/user-attachments/assets/081607cf-ecaa-41be-af03-8d1dfb3e6f35" />

```bash
Ipconfig /release
```

```bash
Ipconfig /renew
```

## CONCLUSIÓN
En conclusión, este proyecto me ha ayudado a entender mejor cómo funciona el protocolo DHCP y los riesgos de seguridad que puede tener si no se protege correctamente. Mediante el ataque de DHCP Starvation, he podido comprobar cómo es posible saturar un servidor DHCP agotando todas las direcciones IP disponibles, lo que provoca que los dispositivos legítimos no puedan conectarse a la red.

Además, he aprendido a utilizar la herramienta Yersinia para realizar ataques de red en un entorno controlado y con fines educativos. También he podido ver la importancia de aplicar medidas de seguridad en el servidor DHCP para reducir el impacto de este tipo de ataques.

En definitiva, este proyecto ha sido muy útil para comprender mejor los ataques en redes y la necesidad de implementar configuraciones seguras para evitar problemas de disponibilidad en los servicios de red.

## BIBLIOGRAFÍA

https://www.prosec-networks.com/en/blog/dhcp-starvation-attack/

https://www.cbtnuggets.com/blog/technology/networking/what-is-a-dhcp-starvation-attack

https://abcxperts.com/que-es-un-ataque-dhcp-starvation/?srsltid=AfmBOooKvQVVhbUA_J6AFTWQiHTGAifhHi-nHrGsrgJbYV4HgbRoKX5I

https://www.youtube.com/watch?v=2JveaM8AHLo

[https://www.youtube.com/watch?v=hx_ScUFb7KI](https://www.youtube.com/watch?v=042ZiAYFIO0)
