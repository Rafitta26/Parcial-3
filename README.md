# Parcial-3

🛡️ Motor de Videojuego 2D - RPG Medieval Arcade
📝 Temática Elegida
Este proyecto implementa la lógica interna de control para un videojuego de estilo RPG Arcade Medieval en cuadrícula 2D (inspirado en mecánicas clásicas como The Legend of Zelda o Gauntlet). El usuario controla a un Héroe que debe explorar un escenario pixelado, interactuar con elementos del mapa (como recolectar monedas), esquivar o combatir orcos enemigos mediante comandos táctiles simulados y sobrevivir gestionando sus puntos de vida.

🏛️ Arquitectura del Software
El diseño del sistema sigue un enfoque minimalista orientado a objetos, desacoplando la gestión de estados, la física espacial y el control de estadísticas para garantizar la escalabilidad futura hacia una interfaz gráfica.

Se han utilizado 5 clases (respetando la restricción máxima de 6):

EstadoJuego (Enum): Define fuertemente los estados globales del motor (MENU, JUGANDO, PAUSA, GAME_OVER), evitando el uso de cadenas o números mágicos.

MotorJuego (Clase Cerebro): Centraliza el bucle de juego (actualizar()), almacena la lista polimórfica de entidades, procesa los inputs táctiles del jugador y ejecuta el detector de colisiones.

EntidadVideojuego (Clase Abstracta Base): Proporciona la infraestructura espacial (x, y, ancho, alto), el estado de salud, la ruta del recurso gráfico y la función matemática de colisión de cajas (AABB).

Jugador y Enemigo (Clases Derivadas): Especializan el comportamiento. Jugador responde a estímulos directos de la pantalla, mientras que Enemigo integra una máquina de estados de Inteligencia Artificial reactiva según proximidad.

SistemaPuntuacion (Clase de Soporte): Encargada de registrar las estadísticas de la partida de forma aislada, acumulando puntos y bajas del jugador.

📊 Diagrama de Clases UML (Mermaid)

classDiagram
    direction T_B

    class EstadoJuego {
        <<enumeration>>
        MENU
        JUGANDO
        PAUSA
        GAME_OVER
    }

    class MotorJuego {
        -EstadoJuego estadoActual
        -List~EntidadVideojuego~ entidades
        -EntidadVideojuego jugador
        +MotorJuego()
        +iniciarPartida() Void
        +pausarReanudar() Void
        +forzarGameOver() Void
        +agregarEntidad(EntidadVideojuego entidad) Void
        +eliminarEntidad(EntidadVideojuego entidad) Void
        +desplazarJugador(String direccion) Void
        +pulsarBotonAccion() Void
        +actualizar() Void
        -procesarColisiones() Void
        -resolverImpacto(EntidadVideojuego e1, EntidadVideojuego e2) Void
        +getEstadoActual() EstadoJuego
        +getJugador() EntidadVideojuego
    }

    class EntidadVideojuego {
        <<abstract>>
        -String nombre
        -int x
        -int y
        -int ancho
        -int alto
        -int vida
        -String sprite
        +EntidadVideojuego(String nombre, int x, int y, int ancho, int alto, int vida, String sprite)
        +actualizar(EntidadVideojuego jugador)* Void
        +colisionaCon(EntidadVideojuego otra) boolean
        +recibirDanio(int cantidad) Void
        +getNombre() String
        +getX() int
        +setX(int x) Void
        +getY() int
        +setY(int y) Void
        +getAncho() int
        +getAlto() int
        +getVida() int
        +setVida(int vida) Void
    }

    class Jugador {
        +Jugador(String nombre, int x, int y)
        +actualizar(EntidadVideojuego jugador) Void
    }

    class Enemigo {
        -String estadoIA
        -int DISTANCIA_PERSEGUIR$
        -int DISTANCIA_ATACAR$
        +Enemigo(String nombre, int x, int y)
        +actualizar(EntidadVideojuego jugador) Void
        +getEstadoIA() String
    }

    class SistemaPuntuacion {
        -int puntos
        -int enemigosEliminados
        +SistemaPuntuacion()
        +sumarPuntos(int cantidad) Void
        +registrarBaja() Void
        +getPuntos() int
    }

    MotorJuego "1" o--> "0..*" EntidadVideojuego : Almacena e itera
    MotorJuego "1" --> "1" EstadoJuego : Controla
    EntidadVideojuego <|-- Jugador : Hereda de
    EntidadVideojuego <|-- Enemigo : Hereda de
    Main ..> MotorJuego : Instancia y simula
    Main ..> SistemaPuntuacion : Registra progresos

🕹️ Diagrama de Casos de Uso UML (Mermaid)
architecture-beta

graph LR
    subgraph Sistema Motor Videojuego
        CU1((CU-01: Iniciar Partida))
        CU2((CU-02: Desplazar Personaje))
        CU3((CU-03: Pulsar Acción))
        CU4((CU-04: Pausar Juego))
    end

    Jugador((Actor: Jugador)) --- CU1
    Jugador --- CU2
    Jugador --- CU3
    Jugador --- CU4

Caso de Uso 1: Iniciar Partida

Campo                                                      Descripción
Nombre                        CU-01 Iniciar Partida
Objetivo                      "Cambiar el estado del motor a activo, limpiar el mapa e inicializar los personajes principales."
Actor Principal               Jugador.
Precondiciones                El motor de juego debe encontrarse en el estado EstadoJuego.MENU o EstadoJuego.GAME_OVER.
Flujo Principal               Paso 1: El jugador interactúa con el botón de inicio.
                              Paso 2: El sistema limpia colecciones previas de entidades.
                              Paso 3: El sistema instancia al Jugador en las coordenadas iniciales básicas.Paso 4: El sistema cambia su estado global a EstadoJuego.JUGANDO.

Flujos Alternativos          "A1: Si el sistema ya está en estado JUGANDO, se descarta la petición y se imprime un log de aviso."
Postcondiciones               El sistema queda listo para ejecutar ciclos activos dentro del método actualizar().
Reglas de Negocio             No se puede iniciar una nueva partida si ya existe una sesión de juego en curso.

Caso de Uso 2: Desplazar Personaje (Input Táctil)

Campo                                                      Descripción
Nombre                         CU-02 Desplazar Personaje
Objetivo                       Modificar las coordenadas espaciales del personaje jugador basándose en un input direccional simulado.
Actor Principal                Jugador.
Precondiciones                 El juego debe encontrarse estrictamente en estado activo (EstadoJuego.JUGANDO).
Flujo Principal                "Paso 1: El jugador desliza el control táctil enviando una dirección (""ARRIBA"", ""ABAJO"", ""IZQUIERDA"" o ""DERECHA"").
                                Paso 2: El sistema valida que la dirección sea correcta.
                                Paso 3: El sistema altera las propiedades x o y de la entidad jugador.
                                Paso 4: El sistema imprime por consola la nueva ubicación lograda."
Flujos Alternativos             "A1: Si el motor está en PAUSA o MENU, el movimiento se ignora por completo de forma segura.A2: Si la dirección enviada no coincide con los parámetros aceptados, se genera un log de error                                   sintáctico."
Postcondiciones                 Las coordenadas del jugador quedan alteradas para el procesamiento del siguiente ciclo de colisiones.
Reglas de Negocio               El desplazamiento se realiza casilla a casilla (valores enteros discretos) para respetar el entorno en cuadrícula.




    
