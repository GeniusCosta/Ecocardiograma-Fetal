# Laudos de Ecocardiografia Fetal — Clínica ENCOR

Sistema **offline** de geração de laudos de ecocardiografia fetal, em arquivo HTML
único (sem dependência de internet em uso local) e também instalável como aplicativo
(PWA) quando publicado via GitHub Pages. Desenvolvido para a Clínica ENCOR
(Barreiras‑BA), em português, com terminologia cardiológica brasileira.

> **Aviso clínico.** Ferramenta de apoio à elaboração de laudos. Não substitui o
> julgamento médico. Os valores de referência e Z‑scores só devem ser interpretados
> dentro das faixas de idade gestacional validadas em cada fonte (ver abaixo).

---

## Visão geral

A interface organiza o exame em quatro abas e gera, em tempo real, uma prévia do
laudo em página A4 pronta para impressão ou exportação em PDF:

1. **Identificação** — gestante, data, convênio, médico solicitante, indicação,
   idade gestacional (semanas + dias).
2. **Biometria / Z‑score** — frequência cardíaca, índice de pulsatilidade do canal
   arterial e o módulo de Z‑score (diâmetros + septo) indexado por idade gestacional.
3. **Descrição** — achados por segmento anatômico, com biblioteca de **20 modelos
   de cardiopatias** que reescrevem as linhas pertinentes da descrição.
4. **Conclusão** — conclusões assertivas, com regras automáticas e ajustes manuais.

## Principais recursos

- **Achados estruturados.** Cada cardiopatia substitui/!acrescenta as linhas certas
  por segmento (situs, conexões, septo, câmaras, valvas, vias de saída, canal
  arterial, ritmo), evitando contradições no texto.
- **Regras automáticas de conclusão.**
  - Frequência cardíaca fetal: < 110 bpm → bradicardia; 110–180 → ritmo normal;
    > 180 → taquicardia (suprimida quando há achado de ritmo específico).
  - Índice de pulsatilidade do canal arterial < 1,9 → restrição (leve/moderada/grave).
- **Módulo de Z‑score por idade gestacional** (ver fontes abaixo), com razões
  automáticas (VD/VE, AP/Ao e istmo/canal) e sinalização de valores fora da faixa.
- **Edição híbrida.** Descrição e conclusão alternam entre modo automático e edição
  livre, preservando o texto digitado.
- **Persistência local** (localStorage) e banco de pacientes no próprio navegador.
- **Exportação em PDF** (html2canvas + jsPDF) e impressão direta.
- **Offline:** o arquivo único funciona sem internet; como PWA (GitHub Pages) fica
  instalável e com cache offline.

## Fontes dos Z‑scores (apenas referências publicadas e citáveis)

O cálculo é `Z = (f(medido) − esperado) / DP`. Nenhum coeficiente é estimado: só
entram estruturas com fonte publicada. Entrada sempre em **mm**.

| Categoria | Estruturas | Fonte | Modelo | Faixa (sem) |
|---|---|---|---|---|
| Anéis valvares | aórtica, pulmonar, mitral, tricúspide | Schneider 2005 | log–log (cm) | 15–39 |
| Grandes vasos | aorta ascendente, tronco pulmonar, aorta descendente | Schneider 2005 | log–log (cm) | 15–39 |
| Ventrículos (DDF) | VE, VD | Schneider 2005 | log–log (cm) | 15–39 |
| Istmo e canal arterial | istmo (3 vasos e traqueia), canal arterial | Pasquini 2007 | log–log (mm) | 18–37 |
| Septo interventricular | espessura diastólica | Gagnon 2016 | linear (mm) | 18–39 |

- Schneider C. et al. *Ultrasound Obstet Gynecol* 2005;26:599‑605 (DOI 10.1002/uog.2597)
- Pasquini L. et al. *Ultrasound Obstet Gynecol* 2007;29:628‑633 (DOI 10.1002/uog.4021)
- Gagnon C. et al. *J Am Soc Echocardiogr* 2016;29:448‑460

> A razão **istmo/canal** (Pasquini) é ≈ 1; valores fora de **0,74–1,23** sinalizam
> suspeita de hipoplasia de arco / coarctação. Estruturas medidas fora da faixa de
> validade da fonte são marcadas como tal (não geram Z‑score).

## Como usar (uso local)

1. Abra o arquivo `index.html` (ou `Sistema_Laudos_EcoFetal_ENCOR.html`) no navegador.
2. Preencha as abas; a prévia A4 atualiza em tempo real.
3. Exporte em PDF ou imprima. Os pacientes ficam salvos no navegador.

## Versão instalável (PWA via GitHub Pages)

A PWA torna o sistema instalável (Windows/macOS/Android) e com cache offline.
**Só funciona via HTTPS** (GitHub Pages); aberto por `file://` continua funcionando
como página, mas não como PWA.

Arquivos necessários, na mesma pasta do repositório:

```
index.html        (o sistema)
manifest.json     (nome, cor, ícone, modo standalone)
sw.js             (service worker: cache offline)
icon.svg          (ícone do app)
```

Trechos a inserir no `index.html`:

```html
<!-- no <head>, após <meta charset> -->
<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#0F5E5A">
<link rel="apple-touch-icon" href="icon.svg">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-title" content="Eco Fetal ENCOR">
```

```html
<!-- antes de </body> -->
<script>
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => navigator.serviceWorker.register('sw.js').catch(() => {}));
}
</script>
```

**Atualizações:** a cada nova publicação, incremente a versão do cache na primeira
linha do `sw.js` (`encor-ecofetal-v1` → `v2` → `v3` …) para descartar o cache antigo.
O `index.html` usa estratégia *network‑first*: com internet, sempre carrega a versão
mais nova; sem internet, usa a última em cache.

## Estrutura do projeto (build)

O HTML final é montado por substituição de marcadores em um *shell*:

```
shell.html        (estrutura, marcadores @@CSS@@ @@TIMB@@ @@SIG@@ @@JS@@)
encor.css         (estilos; cor da marca em --accent)
timbrado.txt      (timbre/cabeçalho)
assinatura.txt    (assinatura digitalizada, base64)
app.js            (motor: abas, achados, Z-score, PDF, persistência)
→ index.html      (arquivo único resultante)
```

Validação usada no desenvolvimento: `node --check app.js` e teste de fumaça em DOM
(jsdom), além de inspeção do PDF renderizado.

## Tecnologias

HTML/CSS/JavaScript puro · html2canvas · jsPDF · Web Storage (localStorage) ·
File System Access API (com *fallback* de download) · Service Worker + Web App Manifest (PWA).

## Pendências / próximos passos

- Ícones `icon-192.png` e `icon-512.png` (acabamento no iOS; Chrome/Edge/Android já
  usam o `icon.svg`).
- Ajustar a cor `theme_color`/ícone para o tom oficial da ENCOR (`--accent` do `encor.css`);
  o `#0F5E5A` atual é um placeholder.
- Opcional: istmo sagital (Pasquini) e comprimentos de via de entrada VE/VD (Schneider).

## Autoria

**Dr. Genius Costa** — Cardiologista, ecocardiografista e eletrofisiologista
(CRM‑BA 23913; RQE 15055 / 15021 / 16630). Clínica ENCOR — Barreiras, Bahia.

---

*Projeto de uso clínico interno. Os valores de referência pertencem aos respectivos
autores/publicações citados.*
</textContent>
