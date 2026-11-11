# 02. Processo: Evolução e Práticas de Engenharia

## 1. Acontecimentos Relevantes da Evolução

### **Acontecimento 1:** Migração e padronização do sistema de build utilizando Meson.
* **Descrição:** O projeto substituiu ferramentas legadas de compilação (como Autotools) pelo **Meson** combinado com **Ninja**, visando otimizar a velocidade de compilação e a legibilidade dos scripts de configuração.
* **Evidência no Repositório:** A presença imediata do arquivo `meson.build` na raiz do repositório e nas subpastas, substituindo arquivos tradicionais como `configure.ac` ou Makefiles complexos manuais.
### **Acontecimento 2:** Ciclo contínuo de atualizações de compatibilidade com novas versões do GTK/GNOME via Pull Requests.
* **Descrição:** O software passa por manutenções e *releases* frequentes para se adaptar às evoluções das bibliotecas do ambiente gráfico GNOME e correções de descontinuações (*deprecations*) da API do GTK.
* **Evidência no Repositório:** O histórico de *releases* e tags no GitHub (aba *Releases*), além de *commits* recorrentes voltados à adaptação para novas versões do ambiente desktop.
### **Acontecimento 3:** Implementação de melhorias incrementais na gestão de coleções de fontes.
* **Descrição:** A gestão de novas funcionalidades e correções de bugs é feita inteiramente de forma aberta, permitindo que colaboradores externos enviem melhorias pontuais.
* **Evidência no Repositório:** A aba de *Pull Requests* e a listagem de contribuidores (*Contributors*) no GitHub, evidenciando o uso de fluxos baseados em ramas de recursos (*feature branches*) e revisão de código.

## 2. Análise Crítica e Relação com Processos de Software

- **Desenvolvimento Incremental:** Evidenciado pelo histórico de *releases* frequentes, entregando valor de forma constante em vez de esperar um ciclo único de lançamento.
- **Reuso:** O software reaproveita amplamente as bibliotecas gráficas nativas do ambiente GNOME (GTK, GLib, Pango), reduzindo o esforço de desenvolvimento do core visual.
- **Práticas Ágeis e desenvolvimento Open Source:** O fluxo de trabalho baseado em issues e pull requests no GitHub reflete princípios ágeis de transparência, colaboração descentralizada e resposta a feedbacks da comunidade de usuários.