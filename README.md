# Simulador de Paginación de Memoria

## Descripción

Este programa simula el mecanismo de paginación de memoria utilizado por los sistemas operativos. Implementa la gestión de memoria física (RAM) y memoria virtual (SWAP), creación y finalización de procesos, y manejo de page faults con política de reemplazo FIFO.

**Autor**: Maximiliano Solorza  
**Curso**: Sistemas Operativos  
**Universidad**: Universidad Diego Portales  
**Fecha**: Diciembre 2025

## Características Implementadas

### 1. Configuración de Memoria
- Entrada del tamaño de memoria física por el usuario
- Generación aleatoria de memoria virtual (1.5x a 4.5x la memoria física)
- Configuración del tamaño de página
- Configuración del rango de tamaño de procesos

### 2. Gestión de Procesos
- Creación de procesos cada 2 segundos con tamaño aleatorio
- Asignación de páginas en RAM o SWAP según disponibilidad
- Seguimiento de tabla de páginas para cada proceso
- Finalización de procesos aleatorios cada 5 segundos (después de 30s)

### 3. Manejo de Page Faults
- Acceso a direcciones virtuales aleatorias cada 5 segundos (después de 30s)
- Detección de page faults cuando la página está en SWAP
- Implementación de política de reemplazo **FIFO (First In, First Out)**
- Intercambio de páginas entre RAM y SWAP

### 4. Condiciones de Término
- El programa termina cuando no hay memoria suficiente para crear un nuevo proceso
- También termina cuando no hay memoria disponible en RAM ni SWAP

## Requisitos

- Sistema operativo UNIX/Linux
- Compilador g++ con soporte para C++11 o superior
- Bibliotecas estándar de C++

## Compilación

Para compilar el programa, abra una terminal en el directorio donde se encuentra el archivo `paging_simulator.cpp` y ejecute:

```bash
g++ -std=c++11 -pthread -o paging_simulator paging_simulator.cpp
```

### Opciones de compilación:
- `-std=c++11`: Especifica el estándar C++11
- `-pthread`: Habilita el soporte para threads (necesario para `std::this_thread`)
- `-o paging_simulator`: Nombre del ejecutable de salida

## Ejecución

Para ejecutar el programa:

```bash
./paging_simulator
```

### Parámetros de Entrada

El programa solicitará los siguientes parámetros:

1. **Tamaño de la memoria física (MB)**: 
   - Ejemplo: 128, 256, 512
   - Recomendado: 128-512 MB

2. **Tamaño de cada página (KB)**:
   - Ejemplo: 4, 8, 16
   - Recomendado: 4 KB (tamaño típico en sistemas reales)

3. **Tamaño mínimo de proceso (MB)**:
   - Ejemplo: 4, 8, 16
   - Debe ser menor que el tamaño máximo

4. **Tamaño máximo de proceso (MB)**:
   - Ejemplo: 32, 64, 128
   - Debe ser mayor que el tamaño mínimo

### Ejemplo de Ejecución

```bash
$ ./paging_simulator
=== SIMULADOR DE PAGINACION DE MEMORIA ===
Sistemas Operativos - Universidad Diego Portales

Ingrese el tamaño de la memoria física (MB): 128
Ingrese el tamaño de cada página (KB): 4
Ingrese el tamaño mínimo de proceso (MB): 4
Ingrese el tamaño máximo de proceso (MB): 32
```

## Funcionamiento del Programa

### Fase 1: Inicialización (0-30 segundos)
1. Se crea la memoria física (RAM) y virtual (SWAP)
2. Cada 2 segundos se crea un nuevo proceso con tamaño aleatorio
3. Las páginas de cada proceso se asignan primero a RAM, y si está llena, a SWAP
4. Se muestra información de cada proceso creado

### Fase 2: Operación Normal (después de 30 segundos)
1. Continúa la creación de procesos cada 2 segundos
2. **Cada 5 segundos** ocurren dos eventos:
   - Se finaliza un proceso aleatorio, liberando sus páginas
   - Se accede a una dirección virtual aleatoria:
     - Si la página está en RAM: acceso exitoso
     - Si la página está en SWAP: se genera un **PAGE FAULT**
       - Se aplica la política **FIFO** para seleccionar una página víctima en RAM
       - Se intercambian las páginas entre RAM y SWAP

### Política de Reemplazo: FIFO (First In, First Out)

La política FIFO selecciona como víctima la página que fue cargada en RAM hace más tiempo:

1. Se busca en RAM la página con el `loadTime` más antiguo
2. Esta página se mueve a SWAP
3. La página solicitada se carga desde SWAP a RAM
4. Se actualizan las tablas de páginas correspondientes

**Ventajas**: Simple de implementar, justa en términos de antigüedad  
**Desventajas**: Puede reemplazar páginas usadas frecuentemente (anomalía de Belady)

## Salida del Programa

El programa muestra:

- **Log de eventos** con timestamp:
  - Creación de procesos
  - Finalización de procesos
  - Accesos a memoria
  - Page faults
  - Intercambios de páginas (swaps)

- **Estado de memoria** cada 5 segundos:
  - Uso de RAM (páginas y porcentaje)
  - Uso de SWAP (páginas y porcentaje)
  - Número de procesos activos
  - Cantidad de page faults
  - Procesos creados y finalizados

## Estructura del Código

### Clases y Estructuras

1. **PageTableEntry**: Representa una entrada en la tabla de páginas
   - `virtualPage`: Número de página virtual
   - `physicalFrame`: Frame físico (en RAM o SWAP)
   - `location`: "RAM" o "SWAP"

2. **Process**: Representa un proceso
   - `pid`: Identificador único
   - `sizeKB`: Tamaño en KB
   - `numPages`: Número de páginas
   - `pageTable`: Tabla de páginas del proceso
   - `creationTime`: Timestamp de creación

3. **Page**: Representa una página en memoria
   - `pid`: Proceso propietario
   - `pageNum`: Número de página
   - `loadTime`: Timestamp de carga (para FIFO)

4. **PagingSimulator**: Clase principal que maneja toda la simulación

### Métodos Principales

- `createProcess()`: Crea un nuevo proceso y asigna sus páginas
- `finishRandomProcess()`: Finaliza un proceso aleatorio
- `accessVirtualAddress()`: Simula acceso a memoria virtual
- `run()`: Loop principal de la simulación

## Características Técnicas

- **Generación aleatoria**: Utiliza `<random>` para generar valores aleatorios seguros
- **Threads**: Usa `<thread>` para manejar delays y timings
- **Timestamps**: Implementa política FIFO con timestamps reales
- **Gestión de memoria**: Uso de punteros y vectores de C++ para simular memoria


## Notas Importantes

- El programa debe ejecutarse en un sistema UNIX (Linux/macOS)
- La simulación termina automáticamente cuando se agota la memoria
- Los timestamps se muestran en formato HH:MM:SS
- La memoria virtual se genera aleatoriamente entre 1.5x y 4.5x la física




### Si el programa termina muy rápido:
- Aumente el tamaño de la memoria física
- Reduzca el tamaño máximo de los procesos
- Aumente el tamaño de página

### Muchos page faults:
- Es normal, indica que hay poca RAM y mucho uso de SWAP
- Aumente la memoria física para reducir page faults
