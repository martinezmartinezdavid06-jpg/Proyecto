# Taller: El Explorador del Tesoro

## Descripción
Simulador de un explorador que busca un tesoro en una cuadrícula. Implementaciones en Java y Python.

## Archivos
- ExploradorTesoro.java — implementación en Java (Greedy y BFS)
- explorador_tesoro.py — implementación en Python (Greedy y BFS)
- resultado_greedy.txt — salida Greedy (generado al ejecutar)
- resultado_bfs.txt — salida BFS (generado al ejecutar)

## Uso
Java:
  javac ExploradorTesoro.java
  java ExploradorTesoro

Python:
  python explorador_tesoro.py

## Decisiones de diseño
- Se implementaron dos estrategias: Greedy (rápida) y BFS (óptima).
- Se marca celdas visitadas para reducir bucles en Greedy.
- Variables con prefijo `MD_` según requerimiento.

## Limitaciones
- Greedy puede fallar en presencia de obstáculos complejos; BFS es robusto.
