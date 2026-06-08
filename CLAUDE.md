# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é

App de **acompanhamento da manutenção de permutadores (trocadores de calor)** durante
uma parada de manutenção na **Refinaria RPBC (Petrobras), unidade UGAV (105 PP)**.
Empresa: C3 Engenharia & Soluções. Idioma: pt-BR.

Acompanha 58 equipamentos ao longo de 14 etapas de serviço (raqueteamento → abertura →
limpeza → inspeção → ... → desraqueteamento), com fotos de evidência, cronograma,
relatórios imprimíveis e leitura de código de barras/QR para localizar equipamentos.

## Arquitetura

**Aplicação single-file, sem build e sem dependências.** Todo o app vive em
`Acompanhamento_Permutadores_UGAV.html` — HTML + CSS + JavaScript embutidos, mais
imagens em base64. Não há `package.json`, bundler, transpilação, testes ou backend.

### Como rodar
Abrir o `.html` diretamente no navegador (duplo clique ou arrastar). Apenas as fontes
(Google Fonts: Oswald/Barlow) precisam de internet; todo o resto é offline.
Não há comando de build, lint ou test.

### Estrutura do arquivo (linhas aproximadas)
- HTML da UI: tela de `#login`, `#app`, modal de detalhe `#modal`, `#scan` (leitor de
  câmera), `#tv` (modo painel), `#printarea` (impressão de etiquetas/relatórios).
- CSS: temas via `body[data-tema=...]` (azul/preto/branco) usando variáveis CSS.
- `<script>` (a partir de ~linha 524) com as constantes de dados grandes e a lógica.

### Constantes de dados (embutidas no `<script>`)
- `DATA` (~2 MB, linha 525): cadastro mestre dos equipamentos.
  `DATA.order` = lista ordenada de TAGs; `DATA.equip[TAG]` = ficha técnica
  (descrição, tipo, material, dimensões, foto base64, e flags `SIM`/`N/A` que
  indicam quais etapas se aplicam: `retub`, `usin`, `subfeixe`, `pint_int`, etc.).
- `SCHED` (linha 530): cronograma — `start`/`finish` e datas previstas por
  equipamento/etapa (`SCHED.prev[TAG][ETAPA]`).
- `HDRBG` / `LOGO` (linhas 526–527): imagem de fundo do cabeçalho e logo, em base64.
- `ETAPAS` / `ABREV` (linhas 528–529): as 14 etapas e suas abreviações.
- `CATS` (linha 679): categorias — `FEIXE U`, `FLUTUANTE/RETO`, `AIR-COOLER`.

### Estado e persistência (localStorage)
Chaves (linha 533): `c3ugav_track_v3` (progresso), `c3ugav_sess` (sessão),
`c3ugav_tema` (tema), `c3ugav_meta_v1` (metas/cronograma).
- `TRACK[TAG]` = `{ etapas: { ETAPA: {prev, real, pct, foto, res, resp, desc} }, foto }`
  — o que o usuário preenche por etapa (datas, % e foto de evidência em base64).
- `META` = `{ inicio, prazo, _seed }`.
- `DATA`/`SCHED` são **fixos no código** (cadastro); `TRACK`/`META` são **dados do
  usuário** persistidos no navegador.
- Import/Export JSON (`exportar`/`importar`, ~linha 1021): formato
  `{app:'C3-UGAV-Permutadores', v:4, meta, data:TRACK}`. O arquivo
  `acompanhamento_UGAV_*.json` neste repositório é um export desse formato.

### Autenticação e permissões
Usuários **fixos no código** (linha 560): `c3visitante`/`12345` (papel `view`) e
`c3plan`/`101520` (papel `edit`). `podeEditar()` (papel `edit`) controla toda escrita;
no modo `view` as ações exibem "Modo visualização: edição bloqueada". É controle de
acesso apenas de UI — sem segurança real (tudo roda no cliente).

### Views (nav, linhas 465–470) e funções de render
- `painel` → `renderPainel()` — Painel de Controle (KPIs, faixa de parada, cards).
- `resumo` → `renderResumo()` — Resumo Geral (matriz por tipo, gráfico).
- `rdo` → `renderRDO()` — Relatório Diário de Obra imprimível (baixas por dia/período).
- `qc` → `renderQC()` — Controle de Qualidade (evidências fotográficas por etapa).
- `etiquetas` → `renderEtq()` — geração/impressão de etiquetas com código.
- Extras: `abrir(tag)` (modal de detalhe do equipamento), `abrirScan()` (câmera +
  `BarcodeDetector` para achar TAG, com busca manual `matchTag`), `tvOpen()` (modo TV).
- `go(v)` troca a view ativa.

### Lógica de domínio (núcleo)
- `aplica(equip, etapa)` → quais etapas valem para um equipamento (via flags `SIM`/`N/A`);
  `etapasDe(tag)` retorna a lista filtrada.
- `progresso(tag)` = média dos `pct` das etapas aplicáveis; `previsto(tag)` = esperado
  pela data; `statusEt`/`statusGeral` classificam (concluído/em dia/atrasado).
- Cronograma: `aplicarCronograma()`/`seedCronograma()` semeiam as datas previstas de
  `SCHED` em `TRACK` na primeira execução (`META._seed`).

## Convenções
- Tudo em **pt-BR**, inclusive nomes de funções e rótulos.
- UI gerada por strings de template literais injetadas via `innerHTML` (sem framework).
- Datas internamente em ISO `YYYY-MM-DD`; exibição via `fmtBR()` (DD/MM/AAAA).
- Edições **devem** passar por `podeEditar()` antes de gravar e chamar `save(...)`.
- Ao editar o `.html` em terminal no Windows, preserve a codificação **UTF-8** (há
  muitos acentos em chaves/labels; salvar como outra codificação corrompe as etapas).

## Cuidados ao editar
- O arquivo é grande por causa das imagens base64 em `DATA`/`HDRBG`/`LOGO` — edite por
  trechos, evite reescrever o arquivo inteiro.
- Mudar nomes em `ETAPAS` quebra a compatibilidade com `TRACK`/JSON já salvos (as chaves
  de `etapas` são os próprios textos das etapas).
- Bump da versão do export (`v`) ao mudar o formato de import/export.
