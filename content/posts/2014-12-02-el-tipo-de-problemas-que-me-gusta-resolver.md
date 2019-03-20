---
title: El tipo de problemas que me gusta resolver
author: Fabrizio
type: post
date: 2014-12-02T18:28:46+00:00
url: /2014/12/el-tipo-de-problemas-que-me-gusta-resolver/
categories:
  - Remoquetes

---
Hoy una compañera me preguntó si podía compartir sus datos de Distill. [Distill][1] es una extensión para Chrome que te alerta cuando detecta cambios en un sitio web. <a href="https://addons.mozilla.org/es/firefox/addon/alertbox/reviews/636322/" target="_blank">No permite</a> exportar información.

Emocionante. Un problema sin solución aparente. Mío.

Todas las aplicaciones guardan sus datos en algún sitio, y una extensión de navegador es una aplicación. Sospeché que los datos de Distill estarían en la carpeta del perfil local de Chrome.

<img class="aligncenter size-full wp-image-244777295" src="https://i1.wp.com/remoquete.com/wp-content/uploads/2014/12/lib.png?resize=558%2C398" alt="lib" width="558" height="398" srcset="https://i1.wp.com/remoquete.com/wp-content/uploads/2014/12/lib.png?w=558 558w, https://i1.wp.com/remoquete.com/wp-content/uploads/2014/12/lib.png?resize=300%2C213 300w, https://i1.wp.com/remoquete.com/wp-content/uploads/2014/12/lib.png?resize=150%2C106 150w, https://i1.wp.com/remoquete.com/wp-content/uploads/2014/12/lib.png?resize=400%2C285 400w, https://i1.wp.com/remoquete.com/wp-content/uploads/2014/12/lib.png?resize=200%2C142 200w" sizes="(max-width: 558px) 100vw, 558px" data-recalc-dims="1" /> 

<p style="text-align: center;">
  <em>¿Dónde rayos está la base de datos de Distill?</em>
</p>

Para descubrir la carpeta exacta tuve que usar <a href="http://www.uderzo.it/main_products/space_sniffer/" target="_blank">SpaceSniffer</a>, una herramienta que representa los archivos con un _treemap_ e ilumina la actividad de lectura y escritura en tiempo real. Mágico.

Al hacer clic en Distill, se iluminó el culpable.

<img class="aligncenter size-full wp-image-244777294" src="https://i0.wp.com/remoquete.com/wp-content/uploads/2014/12/database.gif?resize=609%2C426" alt="database" width="609" height="426" data-recalc-dims="1" /> 

<p style="text-align: center;">
  <em>Tengo un código 9 en el radar, repito, un código 9&#8230;<br /> </em>
</p>

Distill guarda los datos en la subcarpeta _\databases\_ del perfil de usuario. Para quien sabe cómo Chrome organiza las bases de datos de las extensiones tiene sentido. La ruta entera:

<pre class="lang:batch decode:true">C:\Users\Usuario\AppData\Local\Google\Chrome\User Data\Perfil\databases\chrome-extension_inlikjemeeknofckkjolnjbpehgadgge_0</pre>

En el directorio había un archivo sin extensión. ¿Qué rayos podía ser? Al abrirlo con mi <a href="http://mh-nexus.de/en/hxd/" target="_blank">editor favorito</a> confirmé que se trataba de una base de datos SQLite. Bingo. Un misterio menos.

<p style="text-align: center;">
  <img class="aligncenter size-full wp-image-244777291" src="https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-hxd.png?resize=529%2C319" alt="sqlite-hxd" width="529" height="319" srcset="https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-hxd.png?w=529 529w, https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-hxd.png?resize=300%2C180 300w, https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-hxd.png?resize=150%2C90 150w, https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-hxd.png?resize=400%2C241 400w, https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-hxd.png?resize=200%2C120 200w" sizes="(max-width: 529px) 100vw, 529px" data-recalc-dims="1" />
</p>

<p style="text-align: center;">
  <em>Previsible. Las extensiones <a href="https://www.sqlite.org/famous.html">usan bases de datos SQLite</a> para almacenar información</em>
</p>

Ahora tenía la base de datos. Pero ¿cómo iba a unir dos bases de datos SQLite sin pasar por un servidor? Una (larga) búsqueda me condujo a la utilidad <a href="http://www.sqlite.org/download.html" target="_blank">SQLite Shell (sqlite3)</a>.

<img class="aligncenter size-full wp-image-244777292" src="https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-shell.png?resize=677%2C342" alt="sqlite-shell" width="677" height="342" srcset="https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-shell.png?w=677 677w, https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-shell.png?resize=300%2C151 300w, https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-shell.png?resize=150%2C75 150w, https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-shell.png?resize=400%2C202 400w, https://i2.wp.com/remoquete.com/wp-content/uploads/2014/12/sqlite-shell.png?resize=200%2C101 200w" sizes="(max-width: 677px) 100vw, 677px" data-recalc-dims="1" /> 

Por desgracia, <a href="http://www.sqlite.org/cli.html" target="_blank">la documentación oficial</a> es <s>pésima</s> sucinta. Otra (larga) búsqueda y hallé <a href="http://sqlite.1065341.n5.nabble.com/How-do-you-combine-two-SQLite-databases-td19362.html" target="_blank">la respuesta</a>. Es una tontería. Basta con encadenar tres comandos: dos para sacar los _dumps_ y uno para unirlos.

<pre class="lang:sh decode:true">sqlite3 1.db .dump &gt;&gt; tmp.sql
sqlite3 2.db .dump &gt;&gt; tmp.sql
sqlite3 3.db &lt; tmp.sql</pre>

Ejecuté los tres comandos, sobrescribí el archivo original con el fusionado, abrí Chrome y&#8230; ¡aleluya! Distill mostraba los datos de ambos usuarios. Logré el tan ansiado _merge_.

Me lo pasé pipa. ¿Debería preocuparme?

 [1]: https://distill.io/