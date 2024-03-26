# Despliega mi primera aplicación en Azure

## Mi primer despliegue en la nube

# Parte I - Despliegue app React (frontend) en Azure

1) Busca Azure for Students en tu buscador de preferencias e ingresa con el correo institucional.
![image][1]
Como se muestra en la imagen se crea la cuenta con la subscripción de estudiantes.
3) Crea un budget de 1 dólar para la cuenta
![image][2]
Se muestra la creación del budget para la subscripción de un 1 dolar.
# Parte II - Despliegue app web spring MVC (o spring-boot backend)
1) Inicie [Azure Cloud Shell](https://docs.microsoft.com/en-in/azure/cloud-shell/overview) desde el portal. Para implementar en un grupo de recursos, ingrese el siguiente comando
```shell
az group create --name MyResourceGroup --location westus
```
A continuación se muestra el paso a a paso para acceder al Azure Cloud shell
![image][3]
![image][4]
![image][5]
2) Para crear un plan de servicio de aplicaciones (App service plan)
```shell
az appservice plan create --resource-group MyResourceGroup --name MyPlan --sku F1
```
![image][6]
3) Finalmente, cree el servidor MySQL con un nombre de servidor único.
```shell
az account list-locations --query "[].{DisplayName:displayName, Name:name}" -o table # choose region
az configure --defaults location=eastus # set region
az mysql flexible-server create --resource-group MyResourceGroup --name pongaunnombreunico --admin-user mysqldbuser --admin-password P2ssw0rd@123 --sku-name Standard_B1ms
```
![image][7]
![image][8]
![image][9]
![image][10]
> Importante: Introduzca un nombre de servidor SQL único. Dado que el nombre de Azure SQL Server no admite las convenciones de nomenclatura de mayúsculas y minúsculas UPPER / Camel , utilice minúsculas para el valor del campo Nombre del servidor de base de datos. 
4) Navegue hasta el grupo de recursos que ha creado. Debería ver un servidor **Azure Database for MySQL server** aprovisionado. Seleccione el servidor de base de datos.
Se encuentra el servidor en la sección dada.
![image][11]
5) Seleccione **Properties**. Guarde el **Server name** y el **Server admin login name** en un bloc de notas.
Esto se encontró en el apartado de **Overview** de la base de datos.
![image][13]
>  En nuestro caso el server name es **miltongutierrezlopezsqlserver.mysql.database.azure.com** y el server admin login name es **mysqldbuser**
7) Seleccione **Connection security**. Habilite la opción **Allow access to Azure services** y guarde los cambios. Esto proporciona acceso a los servicios de Azure para todas las bases de datos de su servidor MySQL.
Esto se encontró en el apartado de **Redes** o **Networking**.
![image][12]
## Ejercicio 2: actualización de la configuración de la aplicación web
A continuación, navegue hasta la aplicación web que ha creado. Mientras implementa una aplicación Java, debe cambiar el contenedor web de la aplicación web a Apache Tomcat.
1) Seleccione **Configuration**. Establezca **Stack settings** como se muestra en la imagen a continuación y haga clic en Guardar.
![image][15]

2) Seleccione Overview y click en Browse.

![image](https://github.com/PDSW-ECI/labs/assets/4140058/23e96cc7-473c-4457-aa2c-acce5c7b23ee)

3) La página web se verá como la imagen de abajo.

![image](https://github.com/PDSW-ECI/labs/assets/4140058/87db1d63-7179-4ce8-a013-a6a1c06056d8)

A continuación, debe actualizar las cadenas de conexión para que la aplicación web se conecte correctamente a la base de datos. Hay varias formas de hacerlo, pero para los fines de esta práctica de laboratorio, adoptará un enfoque simple actualizándolo directamente en Azure Portal.

4) Desde Azure Portal, seleccione la aplicación web que aprovisionó. Ir a Configuración | Configuración de la aplicación | Cadenas de conexión y haga clic en + Nueva cadena de conexión.

![image][16]


5) En la ventana Agregar/Editar cadena de conexión, agregue una nueva cadena de conexión MySQL con MyDatabase como nombre, pegue la siguiente cadena para el valor y reemplace MySQL Server Name, su nombre de usuario y su contraseña con los valores apropiados. Haga clic en Actualizar.
```java
jdbc:mysql://{MySQL Server Name}:3306/alm?useSSL=true&requireSSL=false&autoReconnect=true&user={your user name}&password={your password}
```
En nuestro caso estas opciones se encontran en el apartado Configuración de variables de entorno > Cadenas de conexión
![image][17]
El resultado de la cadena es el siguiente:
> **jdbc:mysql://{miltongutierrezlopezsqlserver.mysql.database.azure.com}:3306/alm?useSSL=true&requireSSL=false&autoReconnect=true&user={mysqldbuser}&password={P2ssw0rd@123}**
7) Haga clic en Guardar para guardar la cadena de conexión.
> Nota: Las cadenas de conexión configuradas aquí estarán disponibles como variables de entorno, con el prefijo del tipo de conexión para aplicaciones Java (también para aplicaciones PHP, Python y Node). En el archivo src/main/resources/application.properties, recuperamos la cadena de conexión reemplazando el siguiente código:
```java
# ORM
# next line deletes the database on startup or shutdown
# spring.jpa.hibernate.ddl-auto=create-drop
# next line updates the database on startup
spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=${MYSQLCONNSTR_MyDatabase}
#spring.datasource.username=root
#spring.datasource.password=my-secret-pw
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.jpa.show-sql=true
```
Ahora ha instalado y configurado todos los recursos necesarios para implementar y ejecutar la aplicación.

## Ejercicio 3: implementar los cambios en la aplicación web
### Configurar las credenciales de implementación para Azure App Service
Seguimos con las indicaciones para poder configurar y obtener las credenciales de implementación para nuestra app. Para poder realizar esto primer tenemos que activar los servicios de FTP de la webapp.
![image][18] 
### Configurar credenciales de ámbito de usuario
En este caso las modificaciones se realizan en la linea de comandos (CLI) en la nube de Azure.
![image][20]
![image][21]
### Obtenga credenciales de ámbito de aplicación
Esto simplemente se puede obtener en el apartado de vista general de la web app, en la zona de FTP
![image][19]
Ahora para poder crear el archivo jar de la aplicacion tenemos que realizar unas pequeñas modificaciones en el proyecto. En el apartado de application.properties
![image][22]
Por consiguiente se importa el siguente paquete en LabApp7
![image][23]
Y finalizando compilamos el proyecto.
![image][24]
![image][25]
Adicionalmente el resultado se le cambia el nombre a app.jar y se copia al directorio del servidor. En el apartado de la aplicacion en la parte de Startup command se coloca el siguiente comando java -jar /home/site/wwwroot/app.jar --server.port=80
![image][27]
El resultado final lastimosamente fue un error de aplicacion ya que el container de la app no se pudo inicializar y como tal no respondia al ping en el puerto 80 según la revisión de logs 
![image][26]
## Miembros
- Milton Andres Gutierrez Lopez
- Jhon Sebastian Sosa Muñoz


[1]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azuremilton.png
[2]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azuremiltonbudget.png
[3]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azureshell.png
[4]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azureshell1.png
[5]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azureshell2.png
[6]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azureshell3.png
[7]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azureshell4.png
[8]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azureshell5.png
[9]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azureshell6.png
[10]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/azureshell7.png
[11]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/sqlserver.png
[12]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/sqlserver2.png
[13]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/sqlserver1.png
[14]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/webapp1.png
[15]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/webapp2.png
[16]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/webapp3.png
[17]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/webapp4.png
[18]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/ftp1.png
[19]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/ftp2.png
[20]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/ftpCLI1.png
[21]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/ftpCLI2.png
[22]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/appmodificacion1.png
[23]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/appmodificacion2.png
[24]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/appmodificacion3.png
[25]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/appmodificacion4.png
[26]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/webappfinal.png
[27]: https://github.com/MiltonGutierrez/LAB06-CVDS/blob/master/images/filezilla.png
