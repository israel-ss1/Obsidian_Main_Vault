<%*
// =============================================================================
// SCRIPT MESTRE: SELETOR VISUAL (GRID) + GERADOR DE LINK (PADRÃO PONTO)
// =============================================================================

// --- 1. CONFIGURAÇÕES ---
const pastaRaiz = "Notas/Bíblia";
const arquivoCSV = "biblia_acf.csv"; // Ajuste se necessário

// Lista de Livros
const livros = [
    { a: "gn", n: "Gênesis" }, { a: "ex", n: "Êxodo" }, { a: "lv", n: "Levítico" }, { a: "nm", n: "Números" }, { a: "dt", n: "Deuteronômio" },
    { a: "js", n: "Josué" }, { a: "jz", n: "Juízes" }, { a: "rt", n: "Rute" }, { a: "1sm", n: "1 Samuel" }, { a: "2sm", n: "2 Samuel" },
    { a: "1rs", n: "1 Reis" }, { a: "2rs", n: "2 Reis" }, { a: "1cr", n: "1 Crônicas" }, { a: "2cr", n: "2 Crônicas" }, { a: "ed", n: "Esdras" },
    { a: "ne", n: "Neemias" }, { a: "et", n: "Ester" }, { a: "jó", n: "Jó" }, { a: "sl", n: "Salmos" }, { a: "pv", n: "Provérbios" },
    { a: "ec", n: "Eclesiastes" }, { a: "ct", n: "Cânticos" }, { a: "is", n: "Isaías" }, { a: "jr", n: "Jeremias" }, { a: "lm", n: "Lamentações" },
    { a: "ez", n: "Ezequiel" }, { a: "dn", n: "Daniel" }, { a: "os", n: "Oséias" }, { a: "jl", n: "Joel" }, { a: "am", n: "Amós" },
    { a: "ob", n: "Obadias" }, { a: "jn", n: "Jonas" }, { a: "mq", n: "Miqueias" }, { a: "na", n: "Naum" }, { a: "hc", n: "Habacuque" },
    { a: "sf", n: "Sofonias" }, { a: "ag", n: "Ageu" }, { a: "zc", n: "Zacarias" }, { a: "ml", n: "Malaquias" },
    { a: "mt", n: "Mateus" }, { a: "mc", n: "Marcos" }, { a: "lc", n: "Lucas" }, { a: "jo", n: "João" }, { a: "at", n: "Atos" },
    { a: "rm", n: "Romanos" }, { a: "1co", n: "1 Coríntios" }, { a: "2co", n: "2 Coríntios" }, { a: "gl", n: "Gálatas" }, { a: "ef", n: "Efésios" },
    { a: "fp", n: "Filipenses" }, { a: "cl", n: "Colossenses" }, { a: "1ts", n: "1 Tessalonicenses" }, { a: "2ts", n: "2 Tessalonicenses" },
    { a: "1tm", n: "1 Timóteo" }, { a: "2tm", n: "2 Timóteo" }, { a: "tt", n: "Tito" }, { a: "fm", n: "Filemom" }, { a: "hb", n: "Hebreus" },
    { a: "tg", n: "Tiago" }, { a: "1pe", n: "1 Pedro" }, { a: "2pe", n: "2 Pedro" }, { a: "1jo", n: "1 João" }, { a: "2jo", n: "2 João" },
    { a: "3jo", n: "3 João" }, { a: "jd", n: "Judas" }, { a: "ap", n: "Apocalipse" }
];

const sleep = (ms) => new Promise(r => setTimeout(r, ms));

// --- 2. PROCESSAMENTO ---
async function processarSelecao(livroObj, inputRestante) {
    if (!inputRestante) return;

    // Parser do input
    const partes = inputRestante.toLowerCase().trim().split(/\s+/);
    const cap = partes[0];
    const textoVersiculos = partes[1]; // Pode ser undefined

    const abrev = livroObj.a;
    const nomeLivro = livroObj.n;

    // Localizar Pasta (Suporte a "01 Gênesis")
    let pastaDoLivro = "";
    const folderBiblia = app.vault.getAbstractFileByPath(pastaRaiz);
    if (folderBiblia && folderBiblia.children) {
        const pastaEncontrada = folderBiblia.children.find(folder => 
            folder.name.toLowerCase().includes(nomeLivro.toLowerCase())
        );
        pastaDoLivro = pastaEncontrada ? pastaEncontrada.path : `${pastaRaiz}/${nomeLivro}`;
    } else {
        pastaDoLivro = `${pastaRaiz}/${nomeLivro}`;
    }

    const nomeNotaCapitulo = `${nomeLivro} ${cap}`;
    const caminhoNotaFinal = `${pastaDoLivro}/${nomeNotaCapitulo}.md`;

    // Criar Nota (Lazy Loading)
    if (!app.vault.getAbstractFileByPath(caminhoNotaFinal)) {
        if (!app.vault.getAbstractFileByPath(pastaRaiz)) await app.vault.createFolder(pastaRaiz);
        if (!app.vault.getAbstractFileByPath(pastaDoLivro)) await app.vault.createFolder(pastaDoLivro);

        try {
            const fileCSV = app.vault.getAbstractFileByPath(arquivoCSV);
            if (!fileCSV) { new Notice("CSV não encontrado!"); return; }

            const conteudoCSV = await app.vault.read(fileCSV);
            const buscaPrefixo = `${abrev}|${cap}|`;
            const linhas = conteudoCSV.split("\n");
            const versiculosEncontrados = linhas.filter(l => l.startsWith(buscaPrefixo));

            if (versiculosEncontrados.length > 0) {
                let corpoCapitulo = "";
                versiculosEncontrados.forEach(linha => {
                    const colunas = linha.split("|");
                    if(colunas.length >= 4) {
                        corpoCapitulo += `**${colunas[2]}** ${colunas[3].trim()} ^v${colunas[2]}\n\n`;
                    }
                });

                const templateNota = `---
Livro: "[[Fontes/Livros/${nomeLivro}|${nomeLivro}]]"
tags: [biblia]
---

# ${nomeNotaCapitulo}

${corpoCapitulo}`;

                await app.vault.create(caminhoNotaFinal, templateNota);
                new Notice(`Criado: ${nomeNotaCapitulo}`);
                await sleep(800); 
            } else {
                new Notice("Capítulo não encontrado no CSV.");
                return;
            }
        } catch (e) {
            console.error(e);
            return;
        }
    }

    // --- GERAÇÃO DO LINK (Formato Ponto) ---
    let linkFinal = "";
    
    if (!textoVersiculos) {
        linkFinal = `[[${nomeNotaCapitulo}|${nomeLivro} ${cap}]]`;
    } else {
        let ancora = "";
        if (textoVersiculos.includes(",")) {
            ancora = textoVersiculos.split(",")[0].trim();
        } else if (textoVersiculos.includes("-")) {
            ancora = textoVersiculos.split("-")[0].trim();
        } else {
            ancora = textoVersiculos;
        }
        
        // MUDANÇA AQUI: Trocado : por . no texto de exibição
        linkFinal = `[[${nomeNotaCapitulo}#^v${ancora}|${nomeLivro} ${cap}.${textoVersiculos}]]`;
    }

    app.workspace.activeLeaf.view.editor.replaceSelection(linkFinal);
}

// --- 3. MODAL GRID ---
class BibleGridModal extends tp.obsidian.Modal {
    constructor(app, livros) {
        super(app);
        this.livros = livros;
    }

    onOpen() {
        const { contentEl } = this;
        contentEl.createEl("h2", { text: "📖 Selecione o Livro" });

        contentEl.createEl("style", {
            text: `
                .bible-grid-container {
                    display: grid;
                    grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
                    gap: 10px;
                    max-height: 60vh;
                    overflow-y: auto;
                    padding: 5px;
                }
                .bible-book-btn {
                    padding: 10px 5px;
                    text-align: center;
                    background-color: var(--interactive-normal);
                    border: 1px solid var(--background-modifier-border);
                    border-radius: 6px;
                    cursor: pointer;
                    font-size: 0.9em;
                    transition: all 0.2s;
                }
                .bible-book-btn:hover {
                    background-color: var(--interactive-accent);
                    color: var(--text-on-accent);
                    font-weight: bold;
                }
                .bible-search-bar {
                    width: 100%;
                    padding: 10px;
                    margin-bottom: 15px;
                    border-radius: 5px;
                    border: 1px solid var(--background-modifier-border);
                }
            `
        });

        const searchInput = contentEl.createEl("input", {
            type: "text",
            placeholder: "Filtrar livros...",
            cls: "bible-search-bar"
        });

        const gridContainer = contentEl.createDiv({ cls: "bible-grid-container" });

        const renderButtons = (filter = "") => {
            gridContainer.empty();
            const lowerFilter = filter.toLowerCase();

            this.livros.forEach(livro => {
                if (livro.n.toLowerCase().includes(lowerFilter) || livro.a.toLowerCase().includes(lowerFilter)) {
                    const btn = gridContainer.createDiv({ cls: "bible-book-btn" });
                    btn.setText(livro.n);
                    
                    btn.addEventListener("click", async () => {
                        this.close();
                        const restante = await tp.system.prompt(`📖 ${livro.n}: Digite Capítulo e Versículos (ex: 3 13-17)`);
                        if (restante) {
                            await processarSelecao(livro, restante);
                        }
                    });
                }
            });
        };

        renderButtons();
        searchInput.addEventListener("input", (e) => renderButtons(e.target.value));
        searchInput.focus();
    }

    onClose() {
        this.contentEl.empty();
    }
}

new BibleGridModal(app, livros).open();
%>