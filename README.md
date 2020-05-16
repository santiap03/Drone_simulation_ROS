# Drone ROS&Gazebo
Esto es un proyecto para la materia de Robótica dirigido al pregrado de ingeniería electrónica en la Universidad de Antioquia
en Medellín, Colombia. Los integrantes son Santiago Álvarez Pinzón & Ken Owashi Vallejo en supervisión del profesor Andrés
Fernando Pérez González.

Con esta guía, se podrá montar un drone en un ambiente virtual con el cual puede interactuar. El drone tiene la posibilidad
de manejarse por medio de consola a través de unos comandos que se dejan en este trabajo o si se desea un manejo más directo
igualmente se puede controlar usando un control de pc que disponga a la mano. También tiene desarrollado un controlador PID 
para que sea posible que el drone vaya solo hasta un punto deseado, sin embargo no es inteligente, así que no tiene la 
capacidad de evadir obstáculos en el camino. 

## Especificaciones
El trabajo se desarrolló en **Ubuntu 16.04.1 LTS** corriendo como sistema operativo del PC, no en máquina virtual. La versión de
ROS es **Kinetic** y el entorno virtual se ejecuta en **Gazebo7**. Los archivos de Python se corren en versión 2.7.

Ubuntu: http://old-releases.ubuntu.com/releases/16.04.4/

ROS: http://wiki.ros.org/kinetic/Installation/Ubuntu

Gazebo: http://gazebosim.org/tutorials?cat=install&tut=install_ubuntu&ver=7.0

## Ejecutar una simulación
Primero cierre todos las terminales que estén abiertas y abra una nueva. Vamos a crear un espacio de trabajo con el nombre que usted desee `mkdir -p ~/<catkin_workspace>/src` y luego 

**Nota:** En algunos computadores, varios de los comandos mencionados aquí pueden llegar a no correr bien. Si es su caso, siempre que abra una terminal ejecute primero el comando `sudo -s`.

1. Clone el siguiente repositorio: 
 * `cd <catkin_workspace>`
 * `git clone https://github.com/santiap03/Drone_ROS_Gazebo_Parte2 .`
2. Haga `catkin_make` estando en el workspace.
3. Ejecutar `source devel/setup.bash`
4. Instalar las dependencias:
 * `cd src`
 * `sudo apt-get install ros-kinetic-hector-*`
 * `sudo apt-get install ros-kinetic-ardrone-autonomy`
5. `git clone https://github.com/santiap03/Drone_ROS_Gazebo_Parte1 .`
6. `cd ..` y `catkin_make`
7. Salimos del worksapce con `cd` y ejecutamos `gedit .bashrc`. Se va a abrir un documento editable, en el cual nos ubicaremos en la última línea para pegar el siguiente comando **source ~/<catkin_workspace>/devel/setup.bash**

**Guardar y cerrar**

8. Cierre la terminal.

### Simulación con control automático
1. Abra una nueva terminal. Ejecute 
 * `cd <catkin_workspace>`
 * `sudo -s`
 * `roslaunch cvg_sim_gazebo ardrone_testworld.launch`

**Puede tardar un poco en comenzar, pero al menos debe visualizar que Gazebo se está cargando. Al final de la ejecución
debería observar el mundo virtual creado con el drone puesto allí.**

2. Abra otra terminal y haga la siguiente prueba: `sudo -s` y `rostopic pub -1 /ardrone/takeoff std_msgs/Empty`

  * La terminal debe sacar el mensaje: publishing and latching message for 3.0 seconds
  * La terminal que ejecuta el mundo debe sacar el mensaje: Quadrotor takes off!!

**El drone debería despegar y estar volando a una baja altura del suelo. En caso de que siga volando hacia arriba se recomienda desinstalar ROS y volverlo a instalar.**

3. Con el drone ya volando, abra una nueva terminal y ejecute lo siguiente: `sudo -s` y `rosrun datos node.py`. Se trata de una rutina pregrabada para el drone, donde el controlador se encarga de llevarlo hasta distintos puntos deseados.

*empieza a navegar en el plano a través de los puntos de una rutina previamente definida; en cada punto pasa 5 segundos e inmediatamente se orienta hacia la siguiente posición, ubicándose sobre las mesas, rodeando la casa, cruzando el aro y finalmente aterrizando en el helipuerto.*

4. 

### Detalles Extras

**Cámara del drone:** Ejecute el siguiente comando para poder ver la cámara del drone mientras lo vuela `rosrun image_view image_view image:=/ardrone/image_raw`

**Detección de objetos:** Primero instalemos unas dependencias con `sudo apt-get install ros-kinetic-moveit*` y 
`sudo apt-get install ros-kinetic-find-object-2d`

Con la cámara del drone activa, ejecuta `rosrun find_object_2d find_object_2d image:=/ardrone/image_raw`

## Controlar al drone con un control de PC
1. El contenido necesario para esto se instalo en la carpeta devel al clonar el repositorio parte 2.

2. Igual que en pasos anteriores, se va a agregar unos comandos en el bash: `cd` y `gedit .bashrc`

Por favor, copie y pegue las siguientes líneas en la parte final del documento:
 * roscd
 * export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:`pwd`/ardrone_helpers
 * cd ~

**Guardar y cerrar**

3. Abra una nueva terminal y ejecute:
 * `roscd`
 * `sudo -s`
 * `rosmake ardrone_joystick`
 * `rosmake joy`

4. Lo siguiente será configurar el control para que trabaje correctamente. Ejecute en una nueva terminal `roslaunch ardrone_joystick teleop.launch`

Luego en otra terminal corra:
 * `cd ~/drone_ws/devel/ardrone_helpers/ardrone_joystick/src`
 * `sudo gedit main.cpp`

**Se abrirá un archivo .cpp**

Busque la siguiente parte en ese archivo:

        // Mapeo de la velocidad
        float scale = 1;
        twist.linear.x = scale*joy_msg->axes[xx]; // adelante/atras
        twist.linear.y = scale*joy_msg->axes[xx]; // izquierda/derecha
        twist.linear.z = scale*joy_msg->axes[xx]; // arriba/abajo
        twist.angular.z = scale*joy_msg->axes[xx]; // yaw

        // button 10 (L1): dead man switch
        bool dead_man_pressed = joy_msg->buttons.at(xx);

        // button 11 (R1): switch emergeny state
        bool emergency_toggle_pressed = joy_msg->buttons.at(xx);

        // button 0 (select): switch camera mode
        bool cam_toggle_pressed = joy_msg->buttons.at(xx);

Realmente en las **xx** habrán unos números, pero esos números funcionan para el control usado en el trabajo.

Lo que haremos será averiguar qué números van allí para el control que usted está usando.

5. Sin cerrar las cosas del paso 4, abra una terminal y ejecute `rostopic echo /joy`

En este momento debe estar viendo lo siguiente:

       axes: [-0.0, -0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
       buttons: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

Mueva el joystick que quiera usar para ir adelante o atrás haciendo dicho movimiento y dejelo quieto. Observe cuál axes es el que cambió de valor, siendo la primera posición cero y la última como siete. Ese valor es el que va a anotar en `twist.linear.x = scale*joy_msg->axes[xx]; // adelante/atras`

Así sucesivamente va a llenar las **xx** con el axes o buttons que cambie y que ustede desee.

**Nota:** Recuerde que un joystick debería servir para adelante/atrás e izquierda/derecha, y el otro para arriba/abajo y el yaw, si quiere hacerlo similar a un control de drone normal de la vida real. Es importante que configure bien el L1 porque con él es que hará despegar al drone.

Una vez termine con esto, guarde y cierre todo. Vuelva a correr el mundo y ejecute `roslaunch ardrone_joystick teleop.launch`. Ya debería poder manejar con el control a su drone a través del mundo.

## Referencias
* https://github.com/angelsantamaria/tum_simulator/blob/master/README.md
* http://wiki.ros.org/hector_gazebo_plugins
