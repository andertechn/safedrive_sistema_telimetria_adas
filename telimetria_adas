/*
Projeto Safedrive - Telemetria ADAS
Nome: [ANDERSON PAULINO DA SILVA] - RA: [10755805]
Nome: [Anthony Santos da Silva] - RA: [10749316]
*/

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define MAX_AMOSTRAS 100

// Regra A: Pega os 3 sensores e acha o valor do meio
void fusao_sensores(float sensores_frontais[][3], float processamento[][2], int total) {
    int i;
    for (i = 0; i < total; i++) {
        float radar = sensores_frontais[i][0];
        float lidar = sensores_frontais[i][1];
        float camera = sensores_frontais[i][2];
        
        float mediana;
        if ((radar >= lidar && radar <= camera) || (radar >= camera && radar <= lidar)) {
            mediana = radar;
        } else if ((lidar >= radar && lidar <= camera) || (lidar >= camera && lidar <= radar)) {
            mediana = lidar;
        } else {
            mediana = camera;
        }
        
        processamento[i][0] = mediana;
    }
}

// Regra B: Calcula a distancia segura pra nao colidir
void calcular_distancia_segura(float velocidades[][2], float processamento[][2], 
                               int total, float atrito, int sensibilidade) {
    float tempo_reacao;
    
    if (sensibilidade == 1) {
        tempo_reacao = 1.0;
    } else if (sensibilidade == 2) {
        tempo_reacao = 1.5;
    } else {
        tempo_reacao = 2.0;
    }
    
    int i;
    for (i = 0; i < total; i++) {
        float vel_kmh = velocidades[i][0];
        float vel_ms = vel_kmh / 3.6;
        
        float distancia = (vel_ms * tempo_reacao) + (vel_ms * vel_ms) / (2.0 * atrito * 9.81);
        processamento[i][1] = distancia;
    }
}

// Regra C: Checa se vai colidir com o carro da frente
void analise_risco_frontal(float velocidades[][2], float processamento[][2], 
                          int status[][3], int total) {
    int i;
    for (i = 0; i < total; i++) {
        float vel_atual = velocidades[i][0];
        float vel_frente = velocidades[i][1];
        
        if (vel_atual <= vel_frente) {
            status[i][0] = 0;
        } else {
            float distancia_real = processamento[i][0];
            float distancia_segura = processamento[i][1];
            
            if (distancia_real >= distancia_segura) {
                status[i][0] = 0;
            } else if (distancia_real >= distancia_segura * 0.5) {
                status[i][0] = 1;
            } else {
                status[i][0] = 2;
            }
        }
    }
}

// Regra D: Checa se ta perto de colidir com os carros dos lados
void assistente_faixa(float velocidades[][2], float sensores_laterais[][2], 
                      int status[][3], int total) {
    int i;
    for (i = 0; i < total; i++) {
        float vel = velocidades[i][0];
        
        float margem = 0.50;
        if (vel > 80.0) {
            margem = margem + (vel - 80.0) * 0.01;
        }
        
        float esquerda = sensores_laterais[i][0];
        float direita = sensores_laterais[i][1];
        
        if (esquerda < margem) {
            status[i][1] = 2;
        } else if (esquerda < margem + 0.20) {
            status[i][1] = 1;
        } else {
            status[i][1] = 0;
        }
        
        if (direita < margem) {
            status[i][2] = 2;
        } else if (direita < margem + 0.20) {
            status[i][2] = 1;
        } else {
            status[i][2] = 0;
        }
    }
}

// Carrega 50 amostras aleatorias
int carregar_dados_iniciais(float velocidades[][2], float sensores_frontais[][3], 
                            float sensores_laterais[][2]) {
    int i;
    
    for (i = 0; i < 50; i++) {
        velocidades[i][0] = 40.0 + (rand() % 80);
        velocidades[i][1] = 30.0 + (rand() % 80);
        
        sensores_frontais[i][0] = 5.0 + (rand() % 45);
        sensores_frontais[i][1] = 5.0 + (rand() % 45);
        sensores_frontais[i][2] = 5.0 + (rand() % 45);
        
        sensores_laterais[i][0] = 0.2 + (rand() % 15) / 10.0;
        sensores_laterais[i][1] = 0.2 + (rand() % 15) / 10.0;
    }
    
    return 50;
}

// Mostra o relatorio no final
void exibir_relatorio(float velocidades[][2], float sensores_frontais[][3], 
                      float sensores_laterais[][2], float processamento[][2], 
                      int status[][3], int total) {
    int i;
    
    printf("\n========== RELATORIO ==========\n\n");
    
    for (i = 0; i < total; i++) {
        printf("Amostra %d:\n", i + 1);
        
        printf("  Velocidade: %.1f km/h (frente: %.1f km/h)\n", 
               velocidades[i][0], velocidades[i][1]);
        printf("  Sensores: Radar %.2fm, Lidar %.2fm, Camera %.2fm\n", 
               sensores_frontais[i][0], sensores_frontais[i][1], sensores_frontais[i][2]);
        printf("  Faixas: Esq %.2fm, Dir %.2fm\n", 
               sensores_laterais[i][0], sensores_laterais[i][1]);
        
        printf("  Distancia real: %.2fm, Distancia segura: %.2fm\n", 
               processamento[i][0], processamento[i][1]);
        
        printf("  Status frontal: ");
        if (status[i][0] == 0) printf("SEGURO");
        else if (status[i][0] == 1) printf("ATENCAO");
        else printf("RISCO");
        printf("\n");
        
        printf("  Status esquerda: ");
        if (status[i][1] == 0) printf("OK");
        else if (status[i][1] == 1) printf("ATENCAO");
        else printf("PERIGO");
        printf("\n");
        
        printf("  Status direita: ");
        if (status[i][2] == 0) printf("OK");
        else if (status[i][2] == 1) printf("ATENCAO");
        else printf("PERIGO");
        printf("\n");
        
        if (status[i][0] == 2 || status[i][1] == 2 || status[i][2] == 2) {
            printf("  GERAL: CRITICO\n");
        } else if (status[i][0] == 1 || status[i][1] == 1 || status[i][2] == 1) {
            printf("  GERAL: ATENCAO\n");
        } else {
            printf("  GERAL: OK\n");
        }
        
        printf("\n");
    }
}

int main() {
    srand(time(NULL));
    
    float velocidades[MAX_AMOSTRAS][2];
    float sensores_frontais[MAX_AMOSTRAS][3];
    float sensores_laterais[MAX_AMOSTRAS][2];
    float processamento[MAX_AMOSTRAS][2];
    int status[MAX_AMOSTRAS][3];
    
    int total = 0;
    float atrito;
    int sensibilidade;
    int opcao;
    
    printf("Atrito da via: ");
    scanf("%f", &atrito);
    
    printf("Sensibilidade (1=Esportivo, 2=Normal, 3=Seguro): ");
    scanf("%d", &sensibilidade);
    
    opcao = 0;
    while (opcao != 4) {
        printf("\nMENU\n");
        printf("1. Carregar 50 amostras\n");
        printf("2. Inserir uma amostra\n");
        printf("3. Processar e ver relatorio\n");
        printf("4. Sair\n");
        printf("Escolha: ");
        scanf("%d", &opcao);
        
        if (opcao == 1) {
            total = carregar_dados_iniciais(velocidades, sensores_frontais, sensores_laterais);
            printf("Carregadas 50 amostras.\n");
        }
        
        else if (opcao == 2) {
            if (total >= MAX_AMOSTRAS) {
                printf("Limite atingido.\n");
            } else {
                printf("Velocidade atual (km/h): ");
                scanf("%f", &velocidades[total][0]);
                
                printf("Velocidade frente (km/h): ");
                scanf("%f", &velocidades[total][1]);
                
                printf("Radar (m): ");
                scanf("%f", &sensores_frontais[total][0]);
                
                printf("Lidar (m): ");
                scanf("%f", &sensores_frontais[total][1]);
                
                printf("Camera (m): ");
                scanf("%f", &sensores_frontais[total][2]);
                
                printf("Faixa esquerda (m): ");
                scanf("%f", &sensores_laterais[total][0]);
                
                printf("Faixa direita (m): ");
                scanf("%f", &sensores_laterais[total][1]);
                
                total++;
                printf("Amostra adicionada.\n");
            }
        }
        
        else if (opcao == 3) {
            if (total == 0) {
                printf("Nenhuma amostra.\n");
            } else {
                fusao_sensores(sensores_frontais, processamento, total);
                calcular_distancia_segura(velocidades, processamento, total, atrito, sensibilidade);
                analise_risco_frontal(velocidades, processamento, status, total);
                assistente_faixa(velocidades, sensores_laterais, status, total);
                
                exibir_relatorio(velocidades, sensores_frontais, sensores_laterais, 
                                 processamento, status, total);
            }
        }
        
        else if (opcao == 4) {
            printf("Saindo.\n");
        }
        
        else {
            printf("Invalido.\n");
        }
    }
    
    return 0;
}
