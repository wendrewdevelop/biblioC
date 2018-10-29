#include <stdio.h>
#include <stdlib.h>

typedef struct disponivel
{
  int nvezes;
} disponivel;

typedef struct emprestado
{
  int leitor;
  int dia;
  int mes;
  int ano;
} emprestado;

typedef struct livro
{
  int num;
  char tit[50];
  int status;
  char aut[30];
  union
  {
    disponivel disp;
    emprestado empr;
  };
} livro;

FILE *arq;
livro liv;

typedef struct usuario
{
  int num;
  char nom[50];
  char nom2[50];
} usuario;

FILE *arq;
usuario nome;


char opcao_menu()
{
  system("cls");
  printf("\n\n\n\n\n\n\n\n\n\n\n\n");
  printf("                  Bem vindo a biblioteca universitaria dos AMIGOS");
  printf("\n\n\n\n\n\n\n\n\n");
  printf("                            (A)tualizar acervo\n");
  printf("                            (L)istar acervo\n");
  printf("                            (C)adastrar usuario\n");
  printf("                            (U)suarios cadastrados\n");
  printf("                            (E)mprestar livro\n");
  printf("                            (R)eceber livro\n");
  printf("                            (F)im\n");
  printf("                            > ");
  return (toupper(getche()));
}

int encontra_livro(int n)
{
  fread(&liv,sizeof(livro),1,arq);
  while (!feof(arq))
  {
    if (liv.num == n)
    {
      fseek(arq,-sizeof(livro),SEEK_CUR);
      return 1;
    }
    fread(&liv,sizeof(livro),1,arq);
  }
  return 0;
}

void atualizar_acervo()
{
  int num;

  arq = fopen("acervo.dat","a+b");
  if (arq == NULL)
  {
    printf("\nErro ao abrir arquivo\n");
    return;
  }
  printf("\n");
  printf("Numero do livro: ");
  scanf("%d",&num);
  if (encontra_livro(num) == 1)
    printf("Ja existe livro com este numero!\n");
  else
  {
    liv.num = num;
    printf("Titulo do livro: ");
    fflush(stdin);
    gets(liv.tit);
    liv.status = 0;       // livro disponivel
    liv.disp.nvezes = 0;  // numero de emprestimos
    printf("Autor do livro: ");
     fflush(stdin);
    gets(liv.aut);
    fwrite(&liv,sizeof(livro),1,arq);
    printf("Livro %d incorporado ao acervo.\n",num);
  }
  fclose(arq);
}

void listar_acervo()
{
  #define MAX_LINHAS 5
  int lin,pag;

  arq = fopen("acervo.dat","r+b");
  if (arq == NULL)
  {
    printf("\nSem livros no acervo\n");
    return;
  }

  pag = 0;
  lin = MAX_LINHAS;
  fread(&liv,sizeof(livro),1,arq);
  while (!feof(arq))
  {
    if (lin == MAX_LINHAS)
    {
      lin = 0;
      pag++;
      system("Cls");
      printf("                        LISTAGEM DO ACERVO\n\n\n\n\n");

      printf("     +-------+-------------------+---------------+\n");
      printf("     |  NUM  |       TITULO      |     Autor     |\n");
      printf("     +-------+-------------------+---------------+\n\n\n");
    }
    lin++;
    printf(" %05d | %20s | %20s \n",liv.num,liv.tit,liv.aut);

    fread(&liv,sizeof(livro),1,arq);
    if (feof(arq) || (lin == MAX_LINHAS))
    {
      if (!feof(arq))
        system("Pause");
    }
  }
  fclose(arq);
}

int codigo_aluno(int n)
{
  fread(&nome,sizeof(usuario),1,arq);
  while (!feof(arq))
  {
    if (nome.num == n)
    {
      fseek(arq,-sizeof(usuario),SEEK_CUR);
      return 1;
    }
    fread(&nome,sizeof(usuario),1,arq);
  }
  return 0;
}

void cadastrar_usuario()
{
  int num;

  arq = fopen("usuario.dat","a+b");
  if (arq == NULL)
  {
    printf("\nErro ao abrir arquivo\n");
    return;
  }
  printf("\n");
  printf("Codigo Aluno: ");
  scanf("%d",&num);
  if (codigo_aluno(num) == 1)
    printf("Ja existe um usuario com esse codigo!\n");
  else
  {
    nome.num = num;
    printf("    Nome do usuario: ");
    fflush(stdin);
    gets(nome.nom2);
    fwrite(&nome,sizeof(nome),1,arq);
    printf("Usuario cadastrado com sucesso!\n",num);
  }
  fclose(arq);
}

void listar_usuario()
{
  #define MAX_LINHAS 5
  int lin,pag;

  arq = fopen("usuario.dat","r+b");
  if (arq == NULL)
  {
    printf("\nNENHUM USUARIO CADASTRADO\n");
    return;
  }

  pag = 0;
  lin = MAX_LINHAS;
  fread(&nome,sizeof(usuario),1,arq);
  while (!feof(arq))
  {
    if (lin == MAX_LINHAS)
    {
      lin = 0;
      pag++;
      system("Cls");
      printf("                        LISTAGEM DO ACERVO\n\n\n\n\n");

      printf("                   +-------+-------------------+\n");
      printf("                   |  COD  |       NOME        |\n");
      printf("                   +-------+-------------------+\n\n\n");
    }
    lin++;
    printf("\n                  %05d | %20s  \n",nome.num,nome.nom2);

    fread(&nome,sizeof(usuario),1,arq);
    if (feof(arq) || (lin == MAX_LINHAS))
    {
      if (!feof(arq))
        system("Pause");
    }
  }
  fclose(arq);
}

void emprestar()
{
  int num;

  arq = fopen("acervo.dat","r+b");
  if (arq == NULL)
  {
    printf("\nErro ao abrir arquivo\n");
    return;
  }
  printf("\n");
  printf("Numero do livro ..... ");
  scanf("%d",&num);
  if (encontra_livro(num) == NULL)
    printf("Este livro nao existe!\n");
  else
  {
    fread(&liv,sizeof(livro),1,arq);
    if (liv.status == 1)
      printf("Livro ja emprestado (devolucao ate %02d/%02d/%04d)\n",
             liv.empr.dia,liv.empr.mes,liv.empr.ano);
    else
    {
      liv.status = 1;
      liv.empr.leitor = liv.disp.nvezes+1;
      printf("Data de devolucao ... ");
      scanf("%d/%d/%d",&liv.empr.dia,&liv.empr.mes,&liv.empr.ano);
      fseek(arq,-sizeof(livro),SEEK_CUR);
      fwrite(&liv,sizeof(livro),1,arq);
      printf("Emprestimo OK!\n");
    }
  }
  fclose(arq);
}

void receber()
{
  int num;

  arq = fopen("acervo.dat","r+b");
  if (arq == NULL)
  {
    printf("\nErro ao abrir arquivo\n");
    return;
  }
  printf("\n");
  printf("Numero do livro ..... ");
  scanf("%d",&num);
  if (encontra_livro(num) == 0)
    printf("Este livro nao existe!\n");
  else
  {
    fread(&liv,sizeof(livro),1,arq);
    if (liv.status == 0)
      printf("Este livro ja esta disponivel.\n");
    else
    {
      liv.status = 0;
      liv.disp.nvezes = liv.empr.leitor;
      fseek(arq,-sizeof(livro),SEEK_CUR);
      fwrite(&liv,sizeof(livro),1,arq);
      printf("Devolucao OK!\n");
    }
  }
  fclose(arq);
}


int main(int args, char * arg[])
{
  char op;

  do
  {
    op = opcao_menu();
    switch (op)
    {
      case 'A': atualizar_acervo(); break;
      case 'L': listar_acervo(); break;
      case 'C': cadastrar_usuario(); break;
      case 'U': listar_usuario(); break;
      case 'E': emprestar(); break;
      case 'R': receber(); break;
    }
    printf("\n");
    system("Pause");
  }
  while (op != 'F');
  return 0;
}
