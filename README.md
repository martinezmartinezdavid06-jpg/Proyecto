6.1 Java — ExploradorTesoro.java
// ExploradorTesoro.java
// Compilar: javac ExploradorTesoro.java
// Ejecutar: java ExploradorTesoro

import java.io.FileWriter;
import java.io.IOException;
import java.util.*;

class MD_Grid {
    int MD_rows, MD_cols;
    char[][] MD_cells; // '.' vacio, 'E' explorador, 'T' tesoro, '*' visitado

    MD_Grid(int rows, int cols){
        MD_rows = rows; MD_cols = cols;
        MD_cells = new char[rows][cols];
        for(int i=0;i<rows;i++) for(int j=0;j<cols;j++) MD_cells[i][j] = '.';
    }
    boolean MD_isValid(int r,int c){ return r>=0 && r<MD_rows && c>=0 && c<MD_cols; }
    void MD_placeExplorer(int r,int c){ if(MD_isValid(r,c)) MD_cells[r][c] = 'E'; }
    void MD_placeTreasure(int r,int c){ if(MD_isValid(r,c)) MD_cells[r][c] = 'T'; }
    void MD_markVisited(int r,int c){ if(MD_isValid(r,c) && MD_cells[r][c]=='.') MD_cells[r][c] = '*'; }
    void MD_clearCell(int r,int c){ if(MD_isValid(r,c)) MD_cells[r][c] = '.'; }
    void MD_print(){
        for(int i=0;i<MD_rows;i++){
            for(int j=0;j<MD_cols;j++) System.out.print(MD_cells[i][j] + " ");
            System.out.println();
        }
    }
    String MD_snapshot(){
        StringBuilder sb = new StringBuilder();
        for(int i=0;i<MD_rows;i++){
            for(int j=0;j<MD_cols;j++) sb.append(MD_cells[i][j]).append(' ');
            sb.append('\n');
        }
        return sb.toString();
    }
}

class MD_Explorer{
    int MD_r, MD_c;
    int MD_steps;
    List<String> MD_history;
    MD_Explorer(int r,int c){ MD_r=r; MD_c=c; MD_steps=0; MD_history = new ArrayList<>(); MD_history.add(MD_r+","+MD_c); }
    void MD_moveTo(int r,int c){ MD_r=r; MD_c=c; MD_steps++; MD_history.add(MD_r+","+MD_c); }
    int MD_manhattan(int tr,int tc){ return Math.abs(MD_r-tr) + Math.abs(MD_c-tc); }
}

public class ExploradorTesoro{

    static void MD_saveLog(String MD_filename, List<String> MD_lines){
        try(FileWriter fw = new FileWriter(MD_filename)){
            for(String s: MD_lines) fw.write(s + "\n");
        } catch(IOException e){ System.out.println("Error escribiendo archivo: " + e.getMessage()); }
    }

    // Greedy
    public static List<String> MD_runGreedy(int MD_rows, int MD_cols, int MD_er, int MD_ec, int MD_tr, int MD_tc){
        MD_Grid MD_grid = new MD_Grid(MD_rows, MD_cols);
        MD_Explorer MD_ex = new MD_Explorer(MD_er, MD_ec);
        boolean[][] MD_visited = new boolean[MD_rows][MD_cols];

        MD_grid.MD_placeExplorer(MD_er, MD_ec);
        MD_grid.MD_placeTreasure(MD_tr, MD_tc);

        List<String> MD_log = new ArrayList<>();
        MD_log.add("--- Mapa inicial ---");
        MD_log.add(MD_grid.MD_snapshot());

        int[][] MD_dirs = {{-1,0},{1,0},{0,-1},{0,1}}; // up, down, left, right
        String[] MD_dirNames = {"UP","DOWN","LEFT","RIGHT"};

        while(!(MD_ex.MD_r==MD_tr && MD_ex.MD_c==MD_tc)){
            MD_visited[MD_ex.MD_r][MD_ex.MD_c] = true;
            int MD_bestR = MD_ex.MD_r, MD_bestC = MD_ex.MD_c;
            int MD_bestDist = MD_ex.MD_manhattan(MD_tr,MD_tc);
            int MD_choice = -1;
            for(int i=0;i<4;i++){
                int nr = MD_ex.MD_r + MD_dirs[i][0];
                int nc = MD_ex.MD_c + MD_dirs[i][1];
                if(!MD_grid.MD_isValid(nr,nc)) continue;
                if(MD_visited[nr][nc]) continue;
                int dist = Math.abs(nr-MD_tr) + Math.abs(nc-MD_tc);
                if(dist < MD_bestDist){ MD_bestDist = dist; MD_bestR = nr; MD_bestC = nc; MD_choice = i; }
            }
            if(MD_choice==-1){
                for(int i=0;i<4;i++){
                    int nr = MD_ex.MD_r + MD_dirs[i][0];
                    int nc = MD_ex.MD_c + MD_dirs[i][1];
                    if(!MD_grid.MD_isValid(nr,nc)) continue;
                    int dist = Math.abs(nr-MD_tr) + Math.abs(nc-MD_tc);
                    if(dist < MD_bestDist){ MD_bestDist=dist; MD_bestR=nr; MD_bestC=nc; MD_choice = i; }
                }
            }
            if(MD_choice==-1){
                MD_log.add("No se puede avanzar más (posible bloqueo). Terminando.");
                break;
            }

            // Mover
            MD_grid.MD_markVisited(MD_ex.MD_r, MD_ex.MD_c);
            MD_grid.MD_clearCell(MD_ex.MD_r, MD_ex.MD_c);
            MD_ex.MD_moveTo(MD_bestR, MD_bestC);
            if(MD_ex.MD_r==MD_tr && MD_ex.MD_c==MD_tc){
                MD_grid.MD_placeTreasure(MD_tr,MD_tc);
                MD_grid.MD_placeExplorer(MD_ex.MD_r, MD_ex.MD_c);
            } else {
                MD_grid.MD_placeExplorer(MD_ex.MD_r, MD_ex.MD_c);
            }
            MD_log.add("Paso " + MD_ex.MD_steps + ": mover " + MD_dirNames[MD_choice] + " -> (" + MD_ex.MD_r + "," + MD_ex.MD_c + ")");
            MD_log.add(MD_grid.MD_snapshot());
        }

        MD_log.add("--- Resultado ---");
        if(MD_ex.MD_r==MD_tr && MD_ex.MD_c==MD_tc) MD_log.add("Tesoro encontrado en (" + MD_tr + "," + MD_tc + ") despues de " + MD_ex.MD_steps + " pasos.");
        else MD_log.add("Tesoro no encontrado. Pasos dados: " + MD_ex.MD_steps);
        return MD_log;
    }

    // BFS para ruta minima
    public static List<String> MD_runBFS(int MD_rows, int MD_cols, int MD_er, int MD_ec, int MD_tr, int MD_tc){
        MD_Grid MD_grid = new MD_Grid(MD_rows, MD_cols);
        MD_grid.MD_placeTreasure(MD_tr, MD_tc);
        List<String> MD_log = new ArrayList<>();
        MD_log.add("--- Mapa inicial ---");
        MD_grid.MD_placeExplorer(MD_er,MD_ec);
        MD_log.add(MD_grid.MD_snapshot());
        MD_grid.MD_clearCell(MD_er,MD_ec);

        int[][] MD_dirs = {{-1,0},{1,0},{0,-1},{0,1}};
        boolean[][] MD_visited = new boolean[MD_rows][MD_cols];
        int[][] MD_pr = new int[MD_rows][MD_cols];
        int[][] MD_pc = new int[MD_rows][MD_cols];
        for(int i=0;i<MD_rows;i++) for(int j=0;j<MD_cols;j++){ MD_pr[i][j]=-1; MD_pc[i][j]=-1; }

        Queue<int[]> MD_q = new LinkedList<>();
        MD_q.add(new int[]{MD_er,MD_ec}); MD_visited[MD_er][MD_ec] = true;
        boolean MD_found = false;
        while(!MD_q.isEmpty()){
            int[] cur = MD_q.poll();
            int r = cur[0], c = cur[1];
            if(r==MD_tr && c==MD_tc){ MD_found = true; break; }
            for(int[] d: MD_dirs){
                int nr = r + d[0], nc = c + d[1];
                if(nr<0||nr>=MD_rows||nc<0||nc>=MD_cols) continue;
                if(!MD_visited[nr][nc]){ MD_visited[nr][nc] = true; MD_pr[nr][nc]=r; MD_pc[nr][nc]=c; MD_q.add(new int[]{nr,nc}); }
            }
        }

        if(!MD_visited[MD_tr][MD_tc]){ MD_log.add("No hay camino al tesoro."); return MD_log; }

        List<int[]> MD_path = new ArrayList<>();
        int rr = MD_tr, cc = MD_tc;
        while(!(rr==MD_er && cc==MD_ec)){
            MD_path.add(new int[]{rr,cc});
            int pr = MD_pr[rr][cc], pc = MD_pc[rr][cc];
            rr = pr; cc = pc;
        }
        MD_path.add(new int[]{MD_er,MD_ec});
        Collections.reverse(MD_path);

        MD_Explorer MD_ex = new MD_Explorer(MD_er,MD_ec);
        MD_grid.MD_placeExplorer(MD_er,MD_ec);
        MD_log.add("Ruta encontrada (pasos): " + (MD_path.size()-1));
        MD_log.add(MD_grid.MD_snapshot());
        for(int i=1;i<MD_path.size();i++){
            int[] p = MD_path.get(i);
            MD_grid.MD_markVisited(MD_ex.MD_r, MD_ex.MD_c);
            MD_grid.MD_clearCell(MD_ex.MD_r, MD_ex.MD_c);
            MD_ex.MD_moveTo(p[0], p[1]);
            if(MD_ex.MD_r==MD_tr && MD_ex.MD_c==MD_tc){
                MD_grid.MD_placeTreasure(MD_tr,MD_tc);
                MD_grid.MD_placeExplorer(MD_ex.MD_r,MD_ex.MD_c);
            } else {
                MD_grid.MD_placeExplorer(MD_ex.MD_r,MD_ex.MD_c);
            }
            MD_log.add("Paso " + MD_ex.MD_steps + ": -> (" + MD_ex.MD_r + "," + MD_ex.MD_c + ")");
            MD_log.add(MD_grid.MD_snapshot());
        }

        MD_log.add("--- Resultado ---");
        MD_log.add("Tesoro encontrado en (" + MD_tr + "," + MD_tc + ") despues de " + MD_ex.MD_steps + " pasos.");
        return MD_log;
    }

    public static void main(String[] args){
        // Ejemplo: mapa 6x6, explorador en (0,0), tesoro en (4,3)
        int MD_rows = 6, MD_cols = 6;
        int MD_er = 0, MD_ec = 0;
        int MD_tr = 4, MD_tc = 3;

        List<String> MD_log1 = MD_runGreedy(MD_rows,MD_cols,MD_er,MD_ec,MD_tr,MD_tc);
        MD_saveLog("resultado_greedy.txt", MD_log1);
        System.out.println("Greedy terminado. Log guardado en resultado_greedy.txt");

        List<String> MD_log2 = MD_runBFS(MD_rows,MD_cols,MD_er,MD_ec,MD_tr,MD_tc);
        MD_saveLog("resultado_bfs.txt", MD_log2);
        System.out.println("BFS terminado. Log guardado en resultado_bfs.txt");
    }
}
