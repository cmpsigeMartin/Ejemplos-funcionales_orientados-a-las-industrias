# Ejemplo funcional para tomar como base para la creación de sistema de seguimiento de producción.

> ### Aviso.
> Toda la información que se despliegue aca esta extraida de mi software (SIGE) siendo este similar a un software SAP ERP echo a medida para la empresa constructora de estructuras metalicas donde trabajo en la actualidad Pueden obtener mas información acerca de mi software a través de [perfil de LinkedIn](https://www.linkedin.com/in/martin-fernandez-3b56a91a0/) o contactandome a mi E-mail developermartinfernandez@gmail.com . A su vez cabe resaltar que todas las practicas que pongo a prueba son funcionales y capaces de ser usadas para operaciones industriales aunque no recibi educación academica o universitaria por lo cual probablemente se veran practicas no convencionales en el desarrollo del  mismo. Toda critica constructiva para mi evolución como programador es bienvenida.


### Base de datos y esquematización
Para la esquematización de diagramas modelo entidad-relación, creación de procedimientos, vistas y funciones aconsejo usar el software [Dbeaver](https://dbeaver.io/). Para la creación de tablas, conexiones de llaves foraneas, ejecuciónes de scripts prefiero usar [HeidiSQL](https://www.heidisql.com/) Aunque esto ya es una cuestion de gustos. **Todos los diagramas ER que vean a lo largo de esta documentación son extraidos de Dbeaver.**

Lo primero a tener en cuenta es el **reparto de trabajo** que se debe hacer para los distintos sectores. 

**¿Que sector es el encargado de alimentar la base de datos?**

*El sector encargado de esto sera el que inicia la cadena, aquel que (Para una empresa de estructuras) asigna los codigos que llevaran las piezas a lo largo de toda su fabricación hasta llegar a la obra de destino.*

**¿Que sector sera el que manipule la base de datos?**

*Pues el indicado es aquel que tenga mas contacto con las fabriación de las piezas es decir, los supervisores de fabricación ellos son los indicados, a su vez pueden entrar otras areas dependiendo la complejidad y necesidad del cliente como el sector logistico para marcar que la pieza ya ha sido despachada a su destino o el sector de oficinas técnicas donde hace el control de seguimiento de fabricación y deciden que piezas entraran a producción.*

Bien, ya encontramos a los indicados para que manipulen los datos

¿Y ahora?
Ahora toca usar el lapiz y el papel o [esta pagina muy sencilla](dbdesigner.net) para diagramar la base de datos.

Evitando complejidades vamos a un caso sencillo de los datos que necesitamos para realizar la trazabalidad de un pieza metalica.

**Datos**: Obra, Sector, direccion, Codigo, codigo_qr, Descripción, Cantidad, Peso, Longitud, Superficie, operario ensamblador.

Respetando el paradigma de las bases de datos relaciónales no podemos meter toda esta información en una sola pues seria un total desastre, lo correcto seria separar la obra y las piezas y el codigo qr es tablas distintas y a cada una ubicarle los datos que lo determinen 

Obra, sector y dirección tendran su tabla y las columnas seran

 * obra_id
 * obra
 * sector
 * direccion
 
obra_id | obra | sector | direccion
--------|----- | -------|------------
1001 | obra de prueba uno | sector de prueba uno| direccion 123
1002 | obra de prueba dos | sector de prueba dos| direccion 456
 
Codigo, descripción, cantidad, peso, longitud y superficie pueden ir juntas aunque hay que especificar a que obra son designadas.

  * pieza_id
  * obra_id
  * codigo
  * descripcion
  * cantidad
  * peso
  * longitud
  * superficie  
    
  pieza_id | obra_id | codigo | descripcion | cantidad | peso | longitud | superficie 
-----------|---------|------- | ------------|----------|------|--------- | ----------
 1  | 1 | RF1-1 | Pieza S/Dibujo | 3 | 2500 | 9000 | 57
 2  | 2 | RF1-2 | Pieza S/Dibujo | 2 | 1750 | 8500 | 67
 
 Necesitamos una trazabilidad o seguimiento  de estos productos y por ende tambien vamos a necesitar de una tabla mas que separes y trate cada codigo de la misma obra como piezas distintas (ya que en la producción podria fabricarse 1 unidad de la RF1-1 y 1 unidad de la RF1-2 dejando fuera las 2 faltantes de la RF1-1 y la faltante de la RF1-2 de las que tambien necesitamos una trazabalidad ya que el personal asignado para la fabricación puede ser otro, tambien el turno (mañana o tarde), fecha, etc...)
 
 Para ello la proxima tabla generara codigos_qr unicos que lo anexaran con la columna pieza_id de la tabla de piezas.
 
  produccion_id | pieza_id | codigo_qr | operario ensamblador 
----------------|----------|---------- | ---------------------
1 | 1 | RF1-11001-1 | Fulano 
2 | 1 | RF1-11001-2 | Mengano
3 | 1 | RF1-11001-3 | No iniciada
4 | 2 | RF1-21002-1 | No iniciada
5 | 2 | RF1-21002-2 | Zutano

Como se puede observar el codigo qr se compone asi:

[Codigo de pieza][Numero id de obra][y por ultimo un indice que separa en tantas cantidades tenga la pieza] 

¿Como se veria la tabla si no respetariamos el paradigma de base de datos relacional?

obra | sector | direccion | codigo | codigo qr | descripcion | cantidad | largo | peso | superficie | operario ensamblador
---- | ------ | --------- | ------ | --------- | ----------- | -------- | ----- | ---- | -----------| ---------------------
Obra de prueba uno | sector de prueba uno | direccion 123 | RF1-1 | RF1-11001-1 | Pieza S/Dibujo | 3 | 9000 | 2500 | 57 | Fulano
Obra de prueba uno | sector de prueba uno | direccion 123 | RF1-1 | RF1-11001-2 | Pieza S/Dibujo | 3 | 9000 | 2500 | 57 | Mengano
Obra de prueba uno | sector de prueba uno | direccion 123 | RF1-1 | RF1-11001-3 | Pieza S/Dibujo | 3 | 9000 | 2500 | 57 | No iniciada
Obra de prueba dos | sector de prueba dos | direccion 456 | RF1-2 | RF1-21002-1 | Pieza S/Dibujo | 2 | 8500 | 1750 | 67 | No iniciada
Obra de prueba dos | sector de prueba dos | direccion 456 | RF1-2 | RF1-21002-2 | Pieza S/Dibujo | 2 | 8500 | 1750 | 67 | Zutano

Y eso que solo son 11 columnas, imaginese en un proyecto real...Una total locura si se necesita actualizar o manipular muchos datos al mismo tiempo. 

Los diagramas que se veran debajo son extractos de la base de datos que cree para el proposito de trazabalidad.

## Para las piezas

![](https://fotos.subefotos.com/3a402597ed8c29b477c1118a1fc03d1co.jpg)

## Para las obras

![](https://fotos.subefotos.com/5222951a28bc88f8068bde16fc24fdd6o.jpg)

**Se pueden observar muchisimas mas columnas por un echo de necesidades y complejizaciones que existen en los distintos rubros.**

Ahora que tenemos la base de datos con sus tablas, procedimientos de carga, vistas para el usuario, etc...

Queda programar el o los softwares para agregar y manipular los datos. 

En mi caso cada sector de la empresa dispone de sus modulos donde contienen todas las herramientas necesarias para operar, por ejemplo
AutoCAD el software que usan los tecnicos y arquitectos para dibujar las piezas que luego seran nombradas con algunos de los codigos como RF1-1 o RF1-2 expulsa la información de esta forma 

![](https://fotos.subefotos.com/95de2ff52a350ac38a499ff8ea5a92beo.png)

**Mi software toma la información de Excel de esta manera**

![](https://fotos.subefotos.com/deb00401b223f74e081f701ce3275216o.png)

**Y la ubica de esta manera**

Tabla obras 

![](https://fotos.subefotos.com/ccb85c99a4b85d07040259e616632b82o.png)

Tabla de datos estaticos de piezas

![](https://fotos.subefotos.com/94b309cd8cf3c4cc314db7d4ad72952fo.png)

Tabla de desglose 

![](https://fotos.subefotos.com/8845bf8f25e3e2cd1cf7292428dd2fa5o.png)

Tabla de seguimiento de las piezas

![](https://fotos.subefotos.com/5be89f8146aca49d53e79424500ac364o.png)

Vistas de desglose y seguimiento en el software.

![](https://fotos.subefotos.com/de9bf9c1b82130c84577e9e9342f3f9fo.png)

![](https://fotos.subefotos.com/111608301c1a19b2bcfe7e070dd48052o.png)

Y estas son las etiquetas que se pegaran en las piezas y mediante una aplicación tambien creada por mi para dispositivos Android lee codigos qr y actualiza información en la base de datos haciendo que se pueda obtener un seguimiento de producción de la pieza.

![](https://fotos.subefotos.com/b759d609016568cab282349b20c7ab77o.png)






 
