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

Otra opción y posible alternativa a la retirada de CentOS 8, sería migrar los servidores de CentOS a Debian. Aunque es importante tener en cuenta que la migración de un sistema operativo a otro puede ser un proceso complejo que requiere tiempo y recursos.

**Recibimiento de la propuesta:**

El anuncio sobre la retirada de CentOS 8 fue recibido con una mezcla de sorpresa, frustración y desilusión por parte de muchos usuarios. Además, algunos usuarios expresaron su descontento con el hecho de que Red Hat había tomado la decisión sin consultar a la comunidad de usuarios de CentOS, lo que generó un sentimiento de desconfianza y falta de transparencia por parte de Red Hat.

Sin embargo, también hubo algunos usuarios que entendieron la decisión de Red Hat y apreciaron el enfoque en CentOS Stream como una plataforma de desarrollo y pruebas para futuras versiones de RHEL.

En general, la retirada de CentOS 8 del mercado generó una respuesta variada por parte de los usuarios, y sigue siendo un tema de debate en la comunidad de usuarios de sistemas operativos de código abierto.

**Mi opinión:**

Esta decisión tomada por Red Hat no me afecta directamente ya que nunca he trabajado con CentOS, por tanto me puedo adaptar fácilmente a los cambios y a la decisión tomada por Red Hat.

Si bien es cierto, que estas decisiones tomadas por grandes empresas como Red Hat tienen grandes repercusiones para los usuarios y las empresas, los cuales se ven obligados a buscar alternativas, que muchas veces son complejas y costosas.

Respecto a las razones detrás de la decisión de Red Hat, es difícil determinar exactamente cuáles fueron los factores clave que impulsaron su cambio de estrategia. Puede que la decisión de cambiar su estrategia de desarrollo y soporte para CentOS se debiera a que querían enfocarse en ofrecer una versión más cercana a RHEL (con CentOS Stream) o que tuvieran la necesidad de centrarse en nuevos proyectos y tecnologías, o en responder a cambios en el mercado. Pero, a mi parecer, aunque es algo muy cuestionado, a Red Hat no les salía rentable mantener CentOS, ya que el objetivo principal de la empresa al tomar este tipo de decisiones es seguir creciendo, es decir, ganar más dinero, cosa que no hacían con CentOS.

## 2. Crea una cuenta en Red Hat y descárgate la iso de Red Hat Enterprise Linux (RHEL) y evalúa el producto. Comenta el procedimiento de alta.

Para crear una cuenta en Red Hat y descargarse la ISO de Red Hat Enterprise Linux (RHEL), es necesario seguir los siguientes pasos:

1. Visitamos la página web de Red Hat y vamos a ir a la [página para la prueba gratuita de RHEL](https://www.redhat.com/es/technologies/linux-platforms/enterprise-linux). Hacemos click en "Pruébela" y en la siguiente pantalla a "Comience el periodo de prueba".

2. Si tenemos cuenta accedemos. En caso contrario hacemos click en "Register for a Red Hat account". Rellenamos el formulario y creamos nuestra cuenta.

3. Una vez creada nuestra cuenta, iniciamos sesión y vamos al mismo enlace de antes para descargar la prueba gratuita de RHEL. Hacemos click en "Pruébela" y en "Comience el periodo de prueba".

4. Nos abrirá una página con los Términos y condiciones. Aceptamos y continuamos.

5. Comenzará la descarga de la iso de RHEL.

Tras la descarga de la iso, he utilizado esta para instalar RHEL en una máquina virtual. La instalación es muy sencilla.

![instalación](/img/centos/1.png)

![instalación](/img/centos/2.png)

![instalación](/img/centos/3.png)

![instalación](/img/centos/4.png)

Una vez instalado pruebo el sistema, las herramientas y las opciones de configuración que nos ofrece. A simple vista es muy similar a Debian.

![config](/img/centos/5.png)

![config](/img/centos/6.png)

Red Hat utiliza **paquetes RPM** y el solucionador de dependencias **yum**. Pero para tener acceso a los repositorios y poder actualizar y descargar paquetes primero nos pide que nos registremos.

![config](/img/centos/7.png)

![config](/img/centos/8.png)

![config](/img/centos/9.png)

Actualizo el repositorio.

![config](/img/centos/10.png)

![config](/img/centos/11.png)

Tras descargar, instalar y utilizar el producto debo comentar los siguientes aspectos:

- **Respecto al proceso de alta**, en mi caso he utilizado la subscripción de prueba gratuita que dura 60 días, pero si quisiéramos seguir utilizándolo sería necesario adquirir una subscripción o licencia para poder usar RHEL libremente y con el soporte de Red Hat. El proceso de alta para adquirir una licencia de RHEL varía en función de las necesidades y características específicas de cada organización.

- **En cuanto al producto en sí**, RHEL es una distribución de sistema operativo servidor basada en el código fuente de Fedora y desarrollada por Red Hat. Ofrece una gran variedad de herramientas y características para la administración de servidores, como la gestión de usuarios, grupos y permisos, la monitorización de recursos, la virtualización y la automatización de tareas. Además RHEL tiene un alto nivel de estabilidad, seguridad y soporte a largo plazo. Por todo esto entiendo que algunas empresas utilicen este sistema operativo, pero a nivel personal e individual, que requiera de una subscripción, es algo que echa para atrás a muchos usuarios, entre los que me incluyo.

## 3. Descarga la iso de CentOS Stream y evalúa el producto.

Para descargar la iso de CentOS Stream 9 vamos al siguiente [enlace](https://www.centos.org/centos-stream/), seleccionamos la opción que se adapte a nuestro sistema y comenzará la descarga.

Una vez descargado instalamos el sistema operativo en una máquina virtual para probarlo. La instalación es igual y tan sencilla como la de RHEL. Cuando termine accedemos con el usuario creado y comprobamos las herramientas y opciones de configuración que nos ofrece CentOS Stream 9.

![config](/img/centos/12.png)

![config](/img/centos/13.png)

La principal diferencia y ventaja de CentOS Stream en comparación a RHEL es que permite a los usuarios estar al día con las últimas actualizaciones y tecnologías, lo que lo convierte en una opción ideal para aquellos que buscan un sistema operativo servidor actualizado y de vanguardia. También es útil para aquellos que quieren estar al día en el desarrollo de software, ya que CentOS Stream ofrece una vista previa de las características que serán incluidas en las futuras versiones de RHEL, con nuevas funciones y correción de errores. Otras ventajas son:

- Las grandes corporaciones como Facebook usan CentOS Stream y continuamente brindan feedback. Hay un gran feedback por parte de los usuarios y la comunidad, la cual es muy activa.

- Al final, el código que se usa para compilar CentOS Stream sigue siendo el código fuente de RHEL.

- Los desarrolladores pueden corregir errores más rápido incorporando mejoras de código de manera continua.

- CentOS Stream se comporta de la misma manera que CentOS 8. Los usuarios no tienen que aprender a usarlo; funciona perfectamente justo después de la instalación. Todas las aplicaciones, scripts y utilidades funcionan de la misma manera.

- Los paquetes debuginfo y src siempre están disponibles.

Entre las desventajas podríamos nombrar las siguientes:

- CentOS Stream es menos estable que sus predecesores. Esto lo hace inadecuado para los sistemas de producción. Para los usuarios o empresas que quieran estabilidad esta no es una buena opción. Los desarrolladores podrían realizar cambios en el sistema operativo en cualquier momento.

- Los usuarios finales no obtienen soluciones hasta el siguiente punto de lanzamiento, ya que los errores encontrados se deben encontrar, resolver y reciclar en una compilación posterior.

- Los cambios de software y las nuevas características no siempre funcionan bien. Los usuarios deben consultar las notas de la versión y revisar los comentarios de la comunidad antes de decidir actualizar CentOS en su máquina.

- CentOS Stream es un proyecto relativamente nuevo, lo que dificulta predecir su rendimiento a largo plazo.

En resumen, CentOS Stream es una distribución de sistema operativo actualizado y que incluye nuevas funciones adicionales a RHEL. Esta distro es especialmente atractiva para aquellos que buscan estar al día en el desarrollo de software. Además, al ser una distribución gratuita y de código abierto, es una buena opción para aquellos que buscan una alternativa a las distribuciones comerciales del sistema operativo, como RHEL.

## 4. Descarga iso de una de las otras distribuciones candidatas, indica criterios para la elección de la nueva distribución y evalúa el producto.

En mi caso he elegido la opción de Rocky Linux. Para descargarme la iso vamos a la siguiente [página](https://rockylinux.org/download). Comenzamos el proceso de instalación en una máquina virtual.

La instalación es igual que la de RHEL y CentOS Stream. Al hacer uso de Rocky Linux comprobamos que este es una copia exacta de RHEL 9.

![config](/img/centos/16.png)

Rocky Linux fue creada por uno de los creadores de CentOS, Gregory M. Kurtzer, junto con la comunidad de usuarios. El objetivo al crear est proyecto fue el diseño de un sistema operativo de grado empresarial que sería compatible bug-for-bug con RHEL. Es decir, que debería tener los mismos bugs y, por tanto, iguales parches.

La distribución está diseñada para ofrecer una experiencia similar a la de RHEL y CentOS, con un enfoque en la estabilidad y confiabilidad del sistema. Ofrece las mismas herramientas (solo viene con aplicaciones heredadas) de administración de sistemas y configuración, como yum y dnf para la gestión de paquetes, así como herramientas de seguridad y redes como Firewalld y SELinux. También ofrece opciones de configuración avanzadas, permitiendo a los administradores de sistemas personalizar y ajustar el sistema según sus necesidades específicas.

Rocky Linux es útil para empresas y organizaciones que buscan una alternativa estable y de confianza a RHEL y CentOS Linux, pero que no quieren pagar por una licencia. Al igual que CentOS, Rocky Linux está respaldado por una comunidad activa de desarrolladores y usuarios que colaboran en la mejora y desarrollo de la distribución.

En general, Rocky Linux es una distribución sólida y confiable que ofrece una alternativa viable a RHEL y CentOS. La distribución está diseñada para ser fácilmente migrable desde otras distribuciones Linux. Viene con el script "migrate2rocky", lo que permite a los usuarios de CentOS cambiar rápidamente a Rocky Linux y que la transición sea muy sencilla. Además, al ser estable y ofrecer seguridad del sistema, la hace atractiva para empresas y organizaciones que buscan una solución confiable y de bajo costo.

## 5. Instala CentOS 7, y evalúa la herramientas que ofrecen la distribución del punto 3.

CentOS Linux 7 es una distribución de sistema operativo de código abierto basada en el código fuente de RHEL 7. CentOS 7 se centraba en ofrecer una plataforma de servidor estable y segura para la ejecución de aplicaciones empresariales y servicios web. Digo "se centraba" porque, como hemos visto, Red Hat dejó el proyecto de CentOS Linux para centrarse en CentOS Stream. Por lo que esta distro ya no tiene actualizaciones de seguridad.

Vamos a proceder a continuación con la instalación de este sistema operativo. Para ello vamos al siguiente [enlace](https://www.centos.org/download/) para descargar la iso de CentOS 7. En CentOS Linux 7-2009 hacemos click en la arquitectura deseada y se nos abrirá la siguiente [página](http://isoredirect.centos.org/centos/7/isos/x86_64/). De las opciones de descarga que nos presenta cica.es he seleccionado la primera.

Usamos la iso descargada para instalar CentOS 7 en una máquina virtual. Comenzamos el proceso de instalación y vemos que estéticamente la pantalla de instalación de CentOS 7 es un poco diferente a las anteriores.

![config](/img/centos/14.png)

![config](/img/centos/17.png)

![config](/img/centos/15.png)

![config](/img/centos/18.png)

![config](/img/centos/19.png)

![config](/img/centos/20.png)

**Ventajas y desventajas de CentOS 7:**

Tras la instalación y probar un poco el sistema, podemos decir que la ventaja principal que tenía CentOS 7 es que se beneficiaba de la estabilidad y madurez de RHEL, siendo así una distribución muy confiable y adecuada para entornos empresariales críticos.

Además CentOS 7 contaba con una gran comunidad de usuarios y desarrolladores, lo que significaba que hay una amplia variedad de documentación, foros de discusión y recursos disponibles en línea. También, al ser una distribución de código abierto, CentOS 7 es gratuito y puede ser utilizado y distribuido libremente.

Una desventaja de CentOS 7 es que al estar basado en RHEL 7, es menos actualizado en comparación con otras distribuciones más recientes (como son las anteriores). Esto significa que puede no incluir las últimas características y actualizaciones de software. Y también hay que tener en cuenta que al ser una distribución más antigua, es posible que no sea compatible con hardware o software más nuevo.

Por lo general, CentOS 7 es una distribución de sistema operativo confiable y segura, ideal para entornos empresariales que requieren una plataforma estable y que no necesita de las últimas actualizaciones y tecnologías.

**Comparación entre CentOS Stream 9 y CentOS 7:**

CentOS Stream 9 es la versión más reciente de la distribución CentOS Stream, y como tal, presenta algunas diferencias significativas en comparación con CentOS 7.

En primer lugar, CentOS Stream 9 está basado en el código fuente de RHEL 9, mientras que CentOS 7 se basa en RHEL 7, es decir, es una versión más reciente y actual. Esto significa que CentOS Stream 9 ofrece una mayor compatibilidad con las últimas tecnologías y actualizaciones, así como una mejor integración con el ecosistema de herramientas de Red Hat.

Otra diferencia importante es que CentOS Stream 9 ofrece una vista previa de las características y actualizaciones que serán incluidas en futuras versiones de RHEL, mientras que CentOS 7 se centra en ofrecer una versión estable de RHEL 7. Esto significa que CentOS Stream 9 es más adecuado para aquellos que quieren estar al día en el desarrollo de software y desean tener acceso a las últimas características y actualizaciones.

Por otro lado, CentOS Stream 9 es menos estable que CentOS 7, y al ser una distribución más actualizada, puede requerir más recursos de hardware que CentOS 7.

---------------------------------------------------

## Resumen y conclusiones

A continuación hago un resumen de los sistemas vistos anteriormente explicando en qué entornos serían más útiles y haciendo una reflexión final.

- RHEL es una distribución de sistema operativo de pago que se centra en ofrecer una plataforma de servidor estable y segura para la ejecución de aplicaciones empresariales y servicios web. Es ideal para entornos empresariales críticos que requieren soporte técnico y actualizaciones de seguridad de alta calidad y larga duración. Es muy estable y está diseñada para ofrecer un alto nivel de fiabilidad.

- CentOS Linux era una distribución de Linux de código abierto, que se basaba en el mismo código fuente de RHEL.
  - CentOS 7 se basa en el código fuente de RHEL 7. Puede ser útil en entornos empresariales que requieren una plataforma de servidor estable, pero que no necesitan el soporte técnico y las actualizaciones de seguridad ofrecidas por RHEL.
  - CentOS 8 fue retirada y no se puede hacer uso de la distribución.

- CentOS Stream es una distribución de sistema operativo de código abierto basada en el código fuente de RHEL que se utiliza como versión de desarrollo para probar nuevas características y funcionalidades. Es una plataforma más actualizada y moderna con actualizaciones frecuentes, un ciclo de lanzamiento más corto, y es una buena opción para aquellos que buscan una distribución de Linux de alta calidad y que está en constante evolución. Es la distribución ideal para los desarrolladores y usuarios avanzados que desean trabajar con tecnologías más nuevas y experimentar con ellas antes de que se lancen oficialmente en una versión estable.

- Rocky Linux es una distribución de Linux de código abierto que fue creada como una alternativa directa a CentOS Linux después de que Red Hat anunciara que iba a cambiar el modelo de soporte para CentOS. Está diseñado para ser compatible con RHEL y es una buena opción para las organizaciones que buscan un sistema operativo de código abierto estable, seguro y de alta calidad. Es adecuado para entornos de servidor que requieren actualizaciones de seguridad frecuentes y a largo plazo.

Como hemos visto, la retirada de CentOS 8 dejó a muchos usuarios buscando alternativas, pero la existencia de proyectos como Rocky Linux, entre otros, demuestran que hay un gran interés y comunidad detrás de las distribuciones basadas en RHEL.

La elección de la distribución adecuada dependerá de las necesidades y objetivos específicos de cada entorno y proyecto. Es importante evaluar cuidadosamente las características y fortalezas de cada distribución antes de tomar una decisión, y siempre tener en cuenta la comunidad de usuarios y desarrolladores activos detrás de cada distribución para asegurarse de que haya suficiente soporte y recursos disponibles en línea.

---------------------------------------------------
