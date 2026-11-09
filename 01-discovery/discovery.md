# 01. Discovery: Investigação e Contexto

## 1. Identificação Básica do Software
* **Nome do Software:** Font Manager (`FontManager/font-manager`)
* **Versão/Release Analisada:** v0.8.8 
* **Licença:** GNU General Public License v3.0 (GPLv3) 
* **Tecnologias Utilizadas:** Linguagem C, toolkit gráfico GTK 3/4, ecossistema GNOME e sistema de build Meson/Ninja 

## 2. Problema e Público-Alvo
* **Problema que o software resolve:** Em sistemas operacionais Linux baseados em desktops modernos (como o ambiente GNOME), gerenciar fontes tipográficas de forma centralizada — instalar, visualizar previews, organizar em coleções e remover arquivos `.ttf` ou `.otf` — costuma exigir manipulação direta de diretórios ou carece de uma interface unificada para usuários comuns e profissionais.
* **Público-alvo / Principais Usuários:** Designers gráficos, criadores de conteúdo, desenvolvedores front-end e usuários avançados de Linux que necessitam gerenciar grandes volumes de fontes.

## 3. Principais Funcionalidades
* Visualização detalhada de fontes instaladas localmente ou no sistema.
* Renderização de amostras de texto customizáveis para teste tipográfico.
* Instalação e desinstalação simplificada de arquivos de fontes.
* Organização por coleções ou grupos de fontes personalizados.

## 4. Personas e Jobs to Be Done (JTBD)

### Persona 1: Mariana, Designer Gráfica
* **Contexto:** Trabalha rotineiramente com identidade visual e precisa testar rapidamente fontes ornamentais e serifadas antes de fechar um projeto de layout.
* **Job to Be Done (JTBD):** *"Quando estou criando uma identidade visual, quero visualizar amostras de texto customizadas com fontes locais, para poder escolher a tipografia ideal sem poluir o sistema operacional com arquivos desnecessários."*

### Persona 2: Carlos, Desenvolvedor Front-End
* **Contexto:** Instala diversas famílias de fontes monoespçadas para ambientes de desenvolvimento e precisa manter coleções separadas por tipo de projeto.
* **Job to Be Done (JTBD):** *"Quando gerencio múltiplos projetos de desenvolvimento, quero agrupar fontes em coleções lógicas, para ativá-las ou desativá-las de forma ágil conforme a necessidade do projeto."*