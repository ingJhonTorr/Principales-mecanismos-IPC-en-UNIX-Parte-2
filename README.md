<h1 align=center>Principales mecanismos IPC en UNIX Parte 2</h1>
Los Mecanismos de Comunicación entre Procesos (IPC), abreviatura de Inter-Process Communication, son herramientas y técnicas fundamentales que dotan de capacidad a los procesos en sistemas Unix para establecer conexiones y compartir recursos de manera eficiente y colaborativa.

<br><h2 align=center>Cola de Mensajes - Message Queue</h2>
Es un mecanismo de comunicación entre procesos y permite que se envíen mensajes de manera asíncrona a través de la cola. Cada mensaje en la cola tiene un identificador de tipo y un conjunto de datos asociados. Los procesos pueden poner mensajes en la cola y recibir mensajes de ella según el tipo de mensaje que deseen leer.
Las colas de mensajes son especialmente útiles en situaciones donde varios procesos deben compartir información o coordinarse sin la necesidad de una comunicación directa o en tiempo real.
Se presentan dos ejemplos en C que utilizan Colas de Mensajes:

#### **Mensaje Específico entre Procesos con Colas de Mensajes**
Estos dos programas en C muestran una forma de comunicación específica entre procesos utilizando colas de mensajes. El **Proceso1** crea una cola de mensajes y envía un tipo de mensaje específico al **Proceso2**. El **Proceso2** abre la cola de mensajes y recibe únicamente el mensaje con el tipo especificado.

#### **Código del Proceso 1:**

```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/msg.h>
    #include <string.h>
    
    struct mensaje {
        long tipo;
        char texto[100];
    };
    
    int main() {
        key_t clave;
        int id_cola;
        struct mensaje mensaje_enviar;
    
        clave = ftok("/tmp", 'A');
        id_cola = msgget(clave, IPC_CREAT | 0666);
        mensaje_enviar.tipo = 2;
        strcpy(mensaje_enviar.texto, "Hola desde el Proceso 1");
        msgsnd(id_cola, &mensaje_enviar, sizeof(mensaje_enviar.texto), 0);
        printf("Mensaje enviado: %s\n", mensaje_enviar.texto);
    
        return 0;
    }
```

#### **Código del Proceso 2:**

```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/msg.h>
    #include <string.h>
    
    struct mensaje {
        long tipo;
        char texto[100];
    };
    
    int main() {
        key_t clave;
        int id_cola;
        struct mensaje mensaje_recibido;
    
        clave = ftok("/tmp", 'A');
        id_cola = msgget(clave, 0666);
        msgrcv(id_cola, &mensaje_recibido, sizeof(mensaje_recibido.texto),2, 0);
        printf("Soy el Proceso 2\n");
        printf("Mensaje recibido: %s\n", mensaje_recibido.texto);
    
        return 0;
    }
```
<br>

#### **Comunicación Asíncrona entre Procesos con Colas de Mensajes**
Estos dos programas en C ilustran la comunicación asíncrona entre procesos utilizando colas de mensajes en sistemas Unix. El **emisor** crea una cola de mensajes y envía mensajes a la misma. El **receptor** abre la cola de mensajes y recibe los mensajes enviados por el emisor. La comunicación es bidireccional y continua hasta que el usuario decide salir precionando la letra **q**.

#### **Código del Emisor:**

```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <sys/msg.h>
    
    struct mensaje {
        long tipo;
        char texto[100];
    };
    
    int main() {
        key_t clave;
        int id_cola;
        struct mensaje mensaje_enviar;
    
        clave = ftok("/tmp", 'A');
        id_cola = msgget(clave, IPC_CREAT | 0666);
    
        if (id_cola == -1) {
            perror("Error al crear la cola de mensajes");
            exit(1);
        }
    
        while (1) {
            printf("Ingrese un mensaje para enviar (o 'q' para salir): ");
            fgets(mensaje_enviar.texto, sizeof(mensaje_enviar.texto), stdin);
    
            if (mensaje_enviar.texto[0] == 'q')
                break;
    
            mensaje_enviar.tipo = 1;
    
            if (msgsnd(id_cola, &mensaje_enviar, sizeof(mensaje_enviar.texto), 0) == -1) {
                perror("Error al enviar el mensaje");
                exit(1);
            }
        }
    
        // Cerrar la cola de mensajes
        msgctl(id_cola, IPC_RMID, NULL);
    
        return 0;
    }
```

#### **Código del Receptor:**

```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/msg.h>
    
    struct mensaje {
        long tipo;
        char texto[100];
    };
    
    int main() {
        key_t clave;
        int id_cola;
        struct mensaje mensaje_recibido;
    
        clave = ftok("/tmp", 'A');
        id_cola = msgget(clave, 0666);
    
        if (id_cola == -1) {
            perror("Error al abrir la cola de mensajes");
            exit(1);
        }
    
        while (1) {
            if (msgrcv(id_cola, &mensaje_recibido, sizeof(mensaje_recibido.texto), 1, 0) == -1) {
                perror("Error al recibir el mensaje");
                exit(1);
            }
    
            printf("Mensaje recibido: %s", mensaje_recibido.texto);
        }
    
        return 0;
    }
```
