// BolsadeValores CODIGO INICIAL IMPLEMENTADO POR CAIO REIS. 
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define TAM_INICIAL 10

//Desenvolver um mercado Simulado de compra e venda de acoes com 3 títulos por exemplo, (PETR4, VALE5, ITSA4
//  USIMI5, LAME3)

typedef struct operacao{
    struct operacao *proxima;
    char qtd[10];
    char Val[10];
}Opera;

typedef struct Acao{
    char sigla[10];
    Opera *Compra_1;
    Opera *Venda_1;
}Acao;

Acao *lista_din_cabeca;
// variáveis  usadas pela função de redimensionamento da array cabeca ou principal
int tam_lista_cabeca = TAM_INICIAL;
int espaco_Cabeca_Util = 0;


Opera *criar_Opera(){
    Opera *operacao = (Opera*) malloc(sizeof(Opera));
    return operacao;
}

Opera *insere_Opera_1(Opera *Opera_1, char qtd[10], char Val[10]){
    Opera *nova_Opera = criar_Opera();
    strcpy(nova_Opera->qtd, qtd);
    strcpy(nova_Opera->Val, Val);
    if(Opera_1 == NULL){
        Opera_1=nova_Opera;
        nova_Opera->proxima=NULL;
    }
    else{
        if(atof(nova_Opera->Val) > atof(Opera_1->Val)){
            nova_Opera->proxima = Opera_1;
            return nova_Opera;
        }
        Opera *aux=Opera_1;
        while(aux->proxima!=NULL && atof(nova_Opera->Val) < atof(aux->proxima->Val)){
            aux=aux->proxima;
        }
        nova_Opera->proxima=aux->proxima;
        aux->proxima=nova_Opera;
    }
    return Opera_1;
}

void exibir_Operacoes(Opera* Opera_1){
    Opera *aux=Opera_1;
    while(aux!=NULL){
        printf("Val: %s\n", aux->Val);
        printf("qtd: %s\n", aux->qtd);
        aux=aux->proxima;
    }
}

int busca_Indice_Sigla(char *sigla) {
    for(int indice=0;indice<espaco_Cabeca_Util;indice++){
        if(!strcmp(lista_din_cabeca[indice].sigla, sigla)) return indice;
    }
    return -1;
}

void dobra_tam_lista_cabecaSeNecessario() {
    if(tam_lista_cabeca <= espaco_Cabeca_Util){
        tam_lista_cabeca = tam_lista_cabeca * 2;
        lista_din_cabeca = (Acao *) realloc(lista_din_cabeca, tam_lista_cabeca * sizeof(Acao) );
    }
}

int insere_NovAcao_Final_ListaCabeca(char *sigla) {
    dobra_tam_lista_cabecaSeNecessario();
    strcpy(lista_din_cabeca[espaco_Cabeca_Util].sigla, sigla);
    lista_din_cabeca[espaco_Cabeca_Util].Compra_1 = NULL;
    lista_din_cabeca[espaco_Cabeca_Util].Venda_1 = NULL;
    espaco_Cabeca_Util = espaco_Cabeca_Util +1;
    int indice_Nova_Sigla = espaco_Cabeca_Util -1;
    return indice_Nova_Sigla;
}

void salvar_Oferta(char *sigla, const char *tipo, char *Val, char *qtd) {

    int indice_Lista_Cabeca_Sigla = busca_Indice_Sigla(sigla);

    if (indice_Lista_Cabeca_Sigla == -1) {
        indice_Lista_Cabeca_Sigla = insere_NovAcao_Final_ListaCabeca(sigla);
    }


    if (tolower(tipo[0]) == 'c') {
        Opera *Compra_1 = insere_Opera_1(lista_din_cabeca[indice_Lista_Cabeca_Sigla].Compra_1, qtd, Val);
        lista_din_cabeca[indice_Lista_Cabeca_Sigla].Compra_1 = Compra_1;

    } else if (tolower(tipo[0]) == 'v') {
        Opera *Venda_1 = insere_Opera_1(lista_din_cabeca[indice_Lista_Cabeca_Sigla].Venda_1, qtd, Val);
        lista_din_cabeca[indice_Lista_Cabeca_Sigla].Venda_1 = Venda_1;
    } else printf("%s precisa ser compra ou venda", lista_din_cabeca[indice_Lista_Cabeca_Sigla].sigla);
}

void listarOfertas(){
    for(int i = 0; i < espaco_Cabeca_Util; i++){
        printf("Sigla: %s\n", lista_din_cabeca[i].sigla);
        printf("COMPRAS:\n");
        exibir_Operacoes(lista_din_cabeca[i].Compra_1);
        printf("\n--------\n");
        printf("\nVENDAS:\n");
        exibir_Operacoes(lista_din_cabeca[i].Venda_1);
        printf("\n###########\n");
    }
}


void atribuir_Ofertas_Arquivo(){
    char *sigla;
    char *tipo;
    char *Val;
    char *qtd;

    
    FILE *arquivo = fopen("Investimento_Remoto.txt", "wt");
    if (!arquivo){
        arquivo = fopen("Investimento_Remoto.txt", "wt");
        if(!arquivo){
            printf("Impossivel ler o arquivo\n");
            exit(1);
        }
    }
    char linhas[1024];

   
    int linha_atual = 0;

    
    int coluna_atual = 0;

   
    while(fgets(linhas, 1024, arquivo)){
        coluna_atual = 0;
        linha_atual++;
        if(linha_atual == 1) continue;

      
        char *celula = strtok(linhas, ",");

       
        while(celula){
            if(coluna_atual == 0){
                sigla = celula;
            }
            if(coluna_atual == 1){
                tipo = celula;
            }
            if(coluna_atual == 2){
               
                Val = celula;
            }
            if(coluna_atual == 3) {

                qtd = celula;
            }
         
            celula = strtok(NULL, ",");
            coluna_atual++;
        }
        salvar_Oferta(sigla, tipo, Val, qtd);
    }
    fclose(arquivo);
}

void inserirOperaDoUsuario() {
    char sigla[10];
    char tipo[10];
    char Val[10];
    char qtd[10];
    printf("\nQual a sigla da Acao?\n");
    scanf("%s", sigla);
    printf("\nQual o tipo de operacao que deseja realizar? [compra]/[venda]\n");
    scanf("%s", tipo);
    printf("\nQual o Valor por unidade?\n");
    scanf("%s", Val);
    printf("\nQual a quantidade que deseja operar?\n");
    scanf("%s", qtd);
    printf("Vales lidos: %s %s %s %s", sigla, tipo, Val, qtd);
    salvar_Oferta(sigla, tipo, Val, qtd);
}

int pega_Val_Opera(Opera *operacao) {
    if (!operacao) return 0;
    return atoi(operacao->Val);
}

Opera *removeOpera(Opera *operacaoASerRemovida, Opera *Opera_1){
    

    
    if(Opera_1->Val==operacaoASerRemovida->Val && Opera_1->qtd==operacaoASerRemovida->qtd){
        Opera_1 = Opera_1->proxima;
        return Opera_1;
    }
   
    else{
        Opera *operacaoAtual = Opera_1;
        while(operacaoAtual){
            if(operacaoAtual->proxima->Val == operacaoASerRemovida->Val && operacaoAtual->proxima->qtd == operacaoASerRemovida->qtd){
                operacaoAtual->proxima = operacaoAtual->proxima->proxima;
            }
            operacaoAtual = operacaoAtual->proxima;
        }
    }
}

void realizarOpera(Opera *compra, Opera *venda, int indice) {
    
    int qtdCompra = atoi(compra->qtd);
    int qtdVenda = atoi(venda->qtd);
    char qtdAtualizada[10];

    
    if(qtdCompra < qtdVenda){
        sprintf(qtdAtualizada, "%d", qtdVenda - qtdCompra);
        strcpy(venda->qtd , qtdAtualizada);
        lista_din_cabeca[indice].Compra_1 = removeOpera(compra, lista_din_cabeca[indice].Compra_1);
    }
    if (qtdCompra > qtdVenda){
        sprintf(qtdAtualizada, "%d", qtdCompra - qtdVenda);
        strcpy(compra->qtd , qtdAtualizada);
        lista_din_cabeca[indice].Venda_1 = removeOpera(venda, lista_din_cabeca[indice].Venda_1);
    }
    if(qtdCompra == qtdVenda){
        lista_din_cabeca[indice].Compra_1 = removeOpera(compra, lista_din_cabeca[indice].Compra_1);
        lista_din_cabeca[indice].Venda_1 = removeOpera(venda, lista_din_cabeca[indice].Venda_1);
    }
}

void comparaPrecoDaCompraComTodasAsVendas(Opera *compra, Opera *venda, int indice) {
    int Val_Compra = pega_Val_Opera(compra);
    Opera *vendaAtual = venda;
    while(vendaAtual){
        if(atof(vendaAtual->Val) == Val_Compra){
            realizarOpera(compra, venda, indice);
        }
        vendaAtual = vendaAtual->proxima;
    }
}

void realizarOperacoes() {



    for(int i=0; i<espaco_Cabeca_Util; i++){
        Opera *compraAtual = lista_din_cabeca[i].Compra_1;


        while(compraAtual){
            if(lista_din_cabeca[i].Venda_1){

                
                comparaPrecoDaCompraComTodasAsVendas(compraAtual, lista_din_cabeca[i].Venda_1, i);
            }
            compraAtual = compraAtual->proxima;
        }

    }
}

void exibirMenu() {
int opcao;
    while(1) {
        printf("\nEscolha uma das opcoes abaixo:\n");
        printf(" [1] Exibir operacoes:\n");
        printf(" [2] Inserir operacoes:\n");
        printf(" [0] Sair:\n  ");
        scanf("%d", &opcao);
        switch (opcao) {
            case 1:
                realizarOperacoes();
                listarOfertas();
                break;
            case 2:
                inserirOperaDoUsuario();
                break;
            default:
                printf("\nSucesso em seus investimentos!!!!!");
                free(lista_din_cabeca);
                exit(0);
        }
    }
}


int main() {
    lista_din_cabeca = (Acao *) malloc(tam_lista_cabeca * sizeof(Acao));
    atribuir_Ofertas_Arquivo();
    exibirMenu();
    return 0;
}
