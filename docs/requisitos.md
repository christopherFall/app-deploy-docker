#
RI-01
RI-02
RI-03
RI-04
RI-05
Requisito de infraestructura
Los tres servicios se ejecutan en contenedores 
independientes.
Solo el proxy inverso está expuesto al exterior.
Los datos de la base persisten a los reinicios.
La solución opera con 4 GB de memoria RAM.
La configuración sensible no está escrita en el 
código.
Criterio de aceptación
docker compose ps muestra tres servicios en estado 
up.
El servicio de base de datos no tiene sección ports y 
no responde desde el host.
Tras down y up, la información sigue disponible.
docker stats no supera ese consumo con los tres 
servicios arriba.
Las credenciales llegan por variables de entorno; el 
archivo .env no está versionado.
