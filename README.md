# wb33-elk

> Pregunta: Habilita el módulo de Nginx y comprueba que recibes los logs adecuadamente. ¿Qué búsqueda harías en Discover, si únicamente queremos mostrar los logs del módulo Nginx?

	event.module: "nginx"
	

> Pregunta: Inspecciona un poco, crea otra visualización que consideres útil y añade las dos visualizaciones a un Dashboard.
He creado un dashboard con el registro ssh y con las conexines a nginx:

![image](https://user-images.githubusercontent.com/65896169/125494255-1fcf9aa2-22c2-40c6-9600-a0d9770da0f2.png)

## Recolección de eventos remotos
Recupera la máquina de la sesión anterior, instala Filebeat, habilita los modulos de System, Nginx y Mysql y realiza las configuraciones necesarias para que Filebeat envíe dichos logs a nuestro Elasticsearch.
> Pregunta: Facilita las capturas que consideres oportunas que demuestren que estás recibiendo los logs de la máquina remota (Netdata).
Aquí tenemos una captura en la que podemos ver por ejemplo que recibimos los logs de nginx del servidor trouble.devops-alumno08.com

![image](https://user-images.githubusercontent.com/65896169/125494290-65b04ce5-ce2b-4ce1-b8bb-48564e30e97e.png)
![image](https://user-images.githubusercontent.com/65896169/125501358-84008f83-9fb7-48f0-8299-02352c22a1e4.png)
![image](https://user-images.githubusercontent.com/65896169/125501575-0479667d-392a-4da8-89e5-58b1b42e7f44.png)

> Pregunta: Añade los logs de php-fpm
