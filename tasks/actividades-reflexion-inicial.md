# ¿Qué diferencia hay entre que un programa funcione «en mi máquina» y que funcione en un servidor de producción?

Son entornos diferentes, por lo cual, si las disposiciones de ambos son distintas, el programa no funcionaría en el entorno de producción. 

# ¿Qué problemas surgen cuando cada integrante de un equipo instala las dependencias de forma distinta?

Que las funcionalidades, procesos y código que estén enlazados a tales dependencias, se pueden quebrar, lo que produce que el programa no corra y en caso de actualizar las dependencias para generar compatibilidad, también podría arrojar errores.

# ¿Por qué una organización querría empaquetar sus aplicaciones en contenedores en lugar de instalarlas directamente en el servidor?

Para que el programa corra en cualquier entorno o maquina sin ningún problema.

# ¿Qué riesgos tendría desplegar un servicio sin analizar antes sus requisitos de hardware, red y seguridad?

No compatibilidad, consumo excesivo de recursos, crash del programa, mala ejecución de la aplicación, fugas de seguridad, exposición de información delicada.