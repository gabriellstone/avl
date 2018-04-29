#include <stdio.h>
#include <stdlib.h>

struct NO {
	int chave; //guarda o conteudo
	int altura;
	struct NO *esq;
	struct NO *dir;
};

/*arvoreAvl criaNO(int chave){
 arvoreAvl novoNO= (arvoreAvl)malloc(sizeof(arvoreAvl));
 novoNO->esq=NULL;
 novoNO->dir=NULL;
 novoNO->chave=chave;
 novoNO->altura=0;
 return novoNO;
 }*/

typedef struct NO* arvoreAvl;

struct NO* criaNO(int chave) {
	struct NO* novoNO = (struct NO*) malloc(sizeof(struct NO));
	novoNO->esq = NULL;
	novoNO->dir = NULL;
	novoNO->chave = chave;
	novoNO->altura = 0;
	return novoNO;
}

arvoreAvl* criaAvl() {
	arvoreAvl* raiz = (arvoreAvl*) malloc(sizeof(arvoreAvl));
	if (raiz != NULL)
		*raiz = NULL;
	return raiz;
}

void libera_no(struct NO* No){
	if(No == NULL) //se o nó tá null
		return;
	libera_no(No->esq); //percorre recursivamente o nó da esquerda e depois o da direita
	libera_no(No->dir);
	free(No);//voltando de percorrer os nós da esquerda e direita, libera o nó.
	No = NULL;
}

void libera_Avl(arvoreAvl* raiz){
	if(raiz==NULL) ///se a raiz não tá nula
		return;
	libera_no(*raiz); //percorre a arvore e libera cada no(essa função que realmente percorre a arvore)
	free(raiz);
}

int altura_NO(struct NO* no) {
	if (!no)///altura de uma árvore vazia é -1
		return -1;
	else
		return no->altura;
}

int max(int a, int b) {
	if (a > b)
		return a;
	return b;
}

int balanceamento(struct NO* no) {
	return abs(altura_NO(no->esq) - altura_NO(no->dir));
}

void ordemCentral(arvoreAvl *raiz) {
	if (raiz == NULL)
		return;
	if (*raiz != NULL) {
		ordemCentral(&((*raiz)->esq));
		printf(" -- %d: com Altura %d e Fator De Balanceamento %d\n",
				(*raiz)->chave, altura_NO(*raiz), balanceamento(*raiz));
		ordemCentral(&((*raiz)->dir));
	}
}

short consulta(arvoreAvl *raiz, int conteudo){
	if(raiz==NULL)
		return 0;
	struct NO* atual= *raiz;
	while(atual!=NULL){
		if(conteudo==atual->chave)
			return 1;
		else if(conteudo>atual->chave)
			atual=atual->dir;
		else
			atual=atual->esq;
	}
	return 0; ///se 1,  conteudo é encontrado

}

void rotacaoDir(arvoreAvl *A) {
	//printf("\nrotacaoDir\n");
	struct NO *B;
	B = (*A)->esq;
	(*A)->esq = B->dir;
	B->dir = *A;
	(*A)->altura = max(altura_NO((*A)->esq), altura_NO((*A)->dir)) + 1;
	B->altura = max(altura_NO(B->esq), (*A)->altura) + 1;
	*A = B;
}

void rotacaoEsq(arvoreAvl *A) {
	// printf("\nrotacaoEsq\n");
	struct NO *B;
	B = (*A)->dir;
	(*A)->dir = B->esq;
	B->esq = (*A);
	(*A)->altura = max(altura_NO((*A)->esq), altura_NO((*A)->dir)) + 1;
	B->altura = max(altura_NO(B->dir), (*A)->altura) + 1;
	(*A) = B;
}

void rotacaoEsqDir(arvoreAvl *A) {
	rotacaoEsq(&(*A)->esq);
	rotacaoDir(A);
}

void rotacaoDirEsq(arvoreAvl *A) {
	rotacaoDir(&(*A)->dir);
	rotacaoEsq(A);
}

int insere(arvoreAvl *raiz, int conteudo) {
	int resposta;
	/*if(!(raiz)){ ///assim tava encerrando o meu código
	        *raiz=criaNO(conteudo);
	    }*/
	if (*raiz == NULL) {
		struct NO *novo;
		novo = (struct NO*) malloc(sizeof(struct NO));
		if (novo == NULL)
			return 0;


		novo->altura = 0;
		novo->esq = NULL;
		novo->dir = NULL;
		novo->chave = conteudo;
		*raiz = novo;
		return 1;
	}

	struct NO *atual = *raiz;
	if (conteudo < atual->chave) {
		if ((resposta = insere(&(atual->esq), conteudo)) == 1) {
			if (balanceamento(atual) >= 2) {
				if (conteudo < (*raiz)->esq->chave) {
					rotacaoDir(raiz);
				} else {
					rotacaoEsqDir(raiz);
				}
			}
		}
	} else {
		if (conteudo > atual->chave) {
			if ((resposta = insere(&(atual->dir), conteudo)) == 1) {
				if (balanceamento(atual) >= 2) {
					if ((*raiz)->dir->chave < conteudo) {
						rotacaoEsq(raiz);
					} else {
						rotacaoDirEsq(raiz);
					}
				}
			}
		} else {
		}
	}

	atual->altura = max(altura_NO(atual->esq), altura_NO(atual->dir)) + 1;

	return resposta;
}

struct NO* menor(struct NO* atual) {
	struct NO *aux = atual;
	struct NO *outroaux = atual->esq;
	while (outroaux != NULL) {
		aux = outroaux;
		aux = outroaux->esq;
	}
	return aux;
}

int remocao(arvoreAvl *raiz, int conteudo) {
	if (*raiz == NULL) {
		printf("conteudo %d nao existe!!\n", conteudo);
		return 0;
	}

	int res;
	if (conteudo < (*raiz)->chave) {
		if ((res = remocao(&(*raiz)->esq, conteudo)) == 1) {
			if (balanceamento(*raiz) >= 2) {
				if (altura_NO((*raiz)->dir->esq)
						<= altura_NO((*raiz)->dir->dir))
					rotacaoEsq(raiz);
				else
					rotacaoDirEsq(raiz);
			}
		}
	}

	if ((*raiz)->chave < conteudo) {
		if ((res = remocao(&(*raiz)->dir, conteudo)) == 1) {
			if (balanceamento(*raiz) >= 2) {
				if (altura_NO((*raiz)->esq->dir)
						<= altura_NO((*raiz)->esq->esq))
					rotacaoDir(raiz);
				else
					rotacaoEsqDir(raiz);
			}
		}
	}

	if ((*raiz)->chave == conteudo) {
		if (((*raiz)->esq == NULL || (*raiz)->dir == NULL)) {
			struct NO *oldNode = (*raiz);
			if ((*raiz)->esq != NULL)
				*raiz = (*raiz)->esq;
			else
				*raiz = (*raiz)->dir;
			free(oldNode);
		} else {
			struct NO* temp = menor((*raiz)->dir);
			(*raiz)->chave = temp->chave;
			remocao(&(*raiz)->dir, (*raiz)->chave);
			if (balanceamento(*raiz) >= 2) {
				if (altura_NO((*raiz)->esq->dir)
						<= altura_NO((*raiz)->esq->esq))
					rotacaoDir(raiz);
				else
					rotacaoEsqDir(raiz);
			}
		}
		if (*raiz != NULL)
			(*raiz)->altura = max(altura_NO((*raiz)->esq),
					altura_NO((*raiz)->dir)) + 1;
		return 1;
	}

	(*raiz)->altura = max(altura_NO((*raiz)->esq), altura_NO((*raiz)->dir)) + 1;

	return res;
}

int main() {
	arvoreAvl *raiz;

	FILE *arqINSERCAO;
	FILE *arqREMOCAO;

	int escolha;

	arqINSERCAO = fopen("/home/gabrielstone/Desktop/Inserção.txt", "r");
	arqREMOCAO = fopen("/home/gabrielstone/Desktop/Remoção.txt", "r");


	int conteudo;
	raiz = criaAvl();

	printf(" \n1 - Inserir 2 - Buscar  3 - Remover   4 - encerrar \n Digite um número: ");
	scanf("%d",&escolha);
	while(escolha!=4) {

		if(escolha==1){
			while (!feof(arqINSERCAO)) {
				fscanf(arqINSERCAO, "%d", &conteudo);
				insere(raiz, conteudo);
			}
			printf(
					"\n    INSERÇÃO CONCLUÍDA    \n");
			ordemCentral(raiz);
		}
		if(escolha==2){
			while (!feof(arqINSERCAO)) {
				fscanf(arqINSERCAO, "%d", &conteudo);
				insere(raiz, conteudo);
			}

			scanf("%i", &conteudo); //inserir um conteudo
			if(consulta(raiz, conteudo)==1){
				printf("Conteudo encontrado\n");
			}
			else
				printf("Conteudo não encontrado\n");

		}
		if(escolha==3) {
			while (!feof(arqINSERCAO)) {
				fscanf(arqINSERCAO, "%d", &conteudo);
				insere(raiz, conteudo);
			}
			while (!feof(arqREMOCAO)) {
				fscanf(arqREMOCAO, "%d", &conteudo);
				remocao(raiz, conteudo);
			}
			printf(
					"\n   Remoção realizada   \n");
			ordemCentral(raiz);


		}
		printf(" \n1 - Inserir 2 - Buscar  3 - Remover   4 - Encerrar \n Digite um número: ");

		scanf("%d",&escolha);
	}

	libera_Avl(raiz);

}
