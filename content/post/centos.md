---
title: "Migración CentOS"
date: 2023-03-21T08:33:20+01:00
draft: true
tags: ["ASO"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

Debido al anuncio del fin de soporte por parte de Red Hat de Centos8 el pasado 31 de diciembre de 2021, y teniendo en cuentas que el fin de vida de centos 7 está programada para el 30 de junio de 2024. Han salido múltiples distribuciones que cubren el hueco dejado por esta distribución tan extendida y tan usada en el ámbito de servidores.

En la presente práctica, analiza posibles versiones candidatas y opciones desplegadas para la migración de tus servidores CentOS.

El espectro es amplio:

- Cambiar el rumbo a una nueva distribución, debian, opensuse, slakware, etc.

- Soluciones aportadas por Red Hat: Red Hat Enterprise Linux, CentOS Stream.

- Solución aportada por Oracle Linux

- Nuevas distribuciones surgidas para paliar el hueco dejado:

  - AlmaLinux

  - Rocky Linux

  - VZLinux

  - euroLinux

--------------------------------------------------

## 1. Analiza el desencadenante de la retirada de CentOS 8 del mercado. ¿Qué opinión tienes al respecto?

La retirada de CentOS 8 del mercado fue un resultado directo de la decisión tomada por Red Hat de cambiar la estrategia de desarrollo y soporte para CentOS.

Antes de la retirada de CentOS 8, CentOS era conocido como un sistema operativo de código abierto y gratuito, basado en la misma base de código que Red Hat Enterprise Linux (RHEL). CentOS ofrecía actualizaciones de seguridad y corrección de errores para los usuarios de forma gratuita, lo que lo convirtió en una opción popular para aquellos que buscaban una alternativa de bajo costo a RHEL.

CentOS 8 tenía varias ventajas, como su estabilidad y confiabilidad como sistema operativo de servidor, la facilidad de uso gracias a su interfaz de línea de comandos y su compatibilidad con aplicaciones de terceros.

Sin embargo, en diciembre de 2020, Red Hat anunció que estaban cambiando su estrategia de desarrollo y soporte para CentOS.

En lugar de seguir manteniendo una versión gratuita de CentOS, Red Hat anunció que se centrarían en el desarrollo de CentOS Stream, que es una versión más cercana a RHEL y se centra en ofrecer una plataforma de desarrollo y pruebas para futuras versiones de RHEL.

La decisión de Red Hat de retirar CentOS 8 del mercado significó que CentOS 8 ya no recibiría actualizaciones de seguridad y corrección de errores después de diciembre de 2021, lo que llevó a muchos usuarios y empresas a buscar alternativas de sistema operativo para reemplazar CentOS 8, realizando una transición costosa y potencialmente compleja.

Algunas comunidades, a raíz del anuncio de la retirada de CentOS 8, empezaron a crear varias distribuciones basadas en el codigo fuente de este, con el objetivo de proporcionar una alternativa de sistema operativo estable y de confianza para los usuarios que dependían de CentOS 8. Algunas de estas distribuciones son:

- **AlmaLinux**: es una distribución gratuita y de código abierto, basada en el código fuente de RHEL y CentOS, que se centra en la estabilidad y la seguridad. Fue creada por CloudLinux, una empresa que se especializa en la seguridad y la estabilidad de los sistemas operativos de servidores.

- **Rocky Linux**: Es otra distribución de sistema operativo de servidor gratuito y de código abierto, creada por la comunidad de usuarios de CentOS que no estaban contentos con la decisión de Red Hat de retirar CentOS 8. Está diseñada para ser una alternativa de sistema operativo estable y de confianza para CentOS.

- **Oracle Linux**: Oracle Linux es una distribución de sistema operativo de servidor basada en el código fuente de RHEL y CentOS, y es totalmente compatible con los paquetes de software de RHEL y CentOS. Oracle Linux está disponible de forma gratuita y ofrece soporte comercial opcional.

- **Springdale Linux**: Es una distribución de sistema operativo de servidor basada en el código fuente de RHEL y CentOS, que se enfoca en proporcionar un entorno de desarrollo y pruebas estable y confiable. Está respaldado por el Centro de Ciencias de la Información de la Universidad de Princeton.

Cada una de estas distribuciones tiene sus propias características y ventajas, y la elección de una u otra depende de las necesidades y preferencias individuales de cada usuario.

**Recibimiento de la propuesta:**

El anuncio sobre la retirada de CentOS 8 fue recibido con una mezcla de sorpresa, frustración y desilusión por parte de muchos usuarios. Además, algunos usuarios expresaron su descontento con el hecho de que Red Hat había tomado la decisión sin consultar a la comunidad de usuarios de CentOS, lo que generó un sentimiento de desconfianza y falta de transparencia por parte de Red Hat.

Sin embargo, también hubo algunos usuarios que entendieron la decisión de Red Hat y apreciaron el enfoque en CentOS Stream como una plataforma de desarrollo y pruebas para futuras versiones de RHEL.

En general, la retirada de CentOS 8 del mercado generó una respuesta variada por parte de los usuarios, y sigue siendo un tema de debate en la comunidad de usuarios de sistemas operativos de código abierto.

**Mi opinión:**

Esta decisión tomada por Red Hat no me afecta directamente ya que nunca he trabajado con CentOS, por tanto me puedo adaptar fácilmente a los cambios y a la decisión tomada por Red Hat.

Si bien es cierto, que estas decisiones tomadas por grandes empresas como Red Hat tienen grandes repercusiones para los usuarios.

Respecto a las razones detrás de la decisión de Red Hat, es difícil determinar exactamente cuáles fueron los factores clave que impulsaron su cambio de estrategia.

Puede que la decisión de cambiar su estrategia de desarrollo y soporte para CentOS no se debiera necesariamente a que CentOS no era rentable para ellos, sino a que querían enfocarse en ofrecer una versión más cercana a RHEL con CentOS Stream. Pero esto es algo muy cuestionado, porque al final el objetivo principal de la empresa al tomar este tipo de decisiones es seguir creciendo, es decir, ganar más dinero.

Otra opción válida es que la decisión de la empresa se haya basado en otros factores, como la necesidad de centrarse en nuevos proyectos y tecnologías, o en responder a cambios en el mercado.

## 2. Crea una cuenta en Red Hat y descárgate la iso de Red Hat Enterprise Linux (RHEL) y evalúa el producto. Comenta el procedimiento de alta.

Comenta el proceso de crear una cuenta en Red Hat y descargarse la iso de Red Hat Enterprise Linux (RHEL). Comenta el procedimiento de alta. Háblame un poco sobre el producto y evalúalo.



Para crear una cuenta en Red Hat y descargarse la ISO de Red Hat Enterprise Linux (RHEL), es necesario seguir los siguientes pasos:

1. Visitamos la página web de Red Hat y vamos a ir a la (página para la prueba gratuita de RHEL)[https://www.redhat.com/es/technologies/linux-platforms/enterprise-linux]. Hacemos click en "Pruébela" y en la siguiente pantalla a "Comience el periodo de prueba".

2. Si tenemos cuenta accedemos. En caso contrario hacemos click en "Register for a Red Hat account". Rellenamo el formulario y creamos nuestra cuenta.

3. Una vez creada nuestra cuenta, iniciamos sesión y vamos al mismo enlace de antes para descargar la prueba gratuita de RHEL. 
en el sitio web de Red Hat y busca la sección de descargas de RHEL.

4. Selecciona la versión de RHEL que deseas descargar y haz clic en el enlace de descarga para iniciar la descarga de la ISO.

5. Una vez que se haya descargado la ISO, grábala en un DVD o utiliza una herramienta como Rufus para crear una unidad USB de arranque con la ISO.

6. Arranca tu ordenador desde la unidad DVD o USB de arranque y sigue las instrucciones para instalar RHEL en tu equipo.

Respecto al proceso de alta, es importante destacar que Red Hat es una empresa que ofrece una versión comercial de su sistema operativo, por lo que es necesario adquirir una licencia para poder usar RHEL en un entorno empresarial. El proceso de alta para adquirir una licencia de RHEL varía en función de las necesidades y características específicas de cada organización, y puede requerir la asistencia de un representante de ventas de Red Hat.

En cuanto al producto en sí, RHEL es una distribución de sistema operativo de servidor basada en el código fuente de Fedora y desarrollada por Red Hat. Ofrece una gran variedad de herramientas y características para la administración de servidores, como la gestión de usuarios, grupos y permisos, la monitorización de recursos, la virtualización y la automatización de tareas. RHEL también es conocido por su alto nivel de estabilidad, seguridad y soporte a largo plazo.

En términos de evaluación, RHEL es una de las distribuciones de sistema operativo de servidor más utilizadas y reconocidas en el mercado empresarial. Ofrece una amplia gama de características y herramientas de administración, así como un alto nivel de seguridad y estabilidad. Sin embargo, su versión comercial requiere la adquisición de una licencia, lo que puede ser un obstáculo para algunos usuarios. Además, RHEL está enfocado en ofrecer un soporte a largo plazo, lo que puede limitar la disponibilidad de las últimas versiones de algunas herramientas y tecnologías.


## 3. Descarga la iso de CentOS Stream y evalúa el producto.

## 4. Descarga iso de una de las otras distribuciones candidatas, indica criterios para la elección de la nueva distribución y evalúa el producto.

## 5. Instala CentOS 7, y evalúa la herramientas que ofrecen la distribución del punto 3.

---------------------------------------------------
