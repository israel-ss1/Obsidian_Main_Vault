<%*
// =============================================================================
// SCRIPT DE CITAÇÃO BÍBLICA (COM LAZY LOADING E SUPORTE A PASTAS NUMERADAS)
// =============================================================================

// 1. MAPEAMENTO DE LIVROS
const nomesLivros = {
    "gn": "Gênesis", "ex": "Êxodo", "lv": "Levítico", "nm": "Números", "dt": "Deuteronômio",
    "js": "Josué", "jz": "Juízes", "rt": "Rute", "1sm": "1 Samuel", "2sm": "2 Samuel",
    "1rs": "1 Reis", "2rs": "2 Reis", "1cr": "1 Crônicas", "2cr": "2 Crônicas",
    "ed": "Esdras", "ne": "Neemias", "et": "Ester", "jó": "Jó", "sl": "Salmos",
    "pv": "Provérbios", "ec": "Eclesiastes", "ct": "Cânticos", "is": "Isaías",
    "jr": "Jeremias", "lm": "Lamentações", "ez": "Ezequiel", "dn": "Daniel",
    "os": "Oséias", "jl": "Joel", "am": "Amós", "ob": "Obadias", "jn": "Jonas",
    "mq": "Miqueias", "na": "Naum", "hc": "Habacuque", "sf": "Sofonias",
    "ag": "Ageu", "zc": "Zacarias", "ml": "Malaquias",
    "mt": "Mateus", "mc": "Marcos", "lc": "Lucas", "joao": "João", "jo": "João", "at": "Atos",
    "rm": "Romanos", "1co": "1 Coríntios", "2co": "2 Coríntios", "gl": "Gálatas",
    "ef": "Efésios", "fp": "Filipenses", "cl": "Colossenses", "1ts": "1 Tessalonicenses",
    "2ts": "2 Tessalonicenses", "1tm": "1 Timóteo", "2tm": "2 Timóteo", "tt": "Tito",
    "fm": "Filemom", "hb": "Hebreus", "tg": "Tiago", "1pe": "1 Pedro", "2pe": "2 Pedro",
    "1jo": "1 João", "2jo": "2 João", "3jo": "3 João", "jd": "Judas", "ap": "Apocalipse"
};

const pastaRaiz = "Notas/Bíblia";
// ATENÇÃO: Ajuste este caminho se o CSV estiver na raiz ou dentro de Notas/Bíblia
const arquivoCSV = "biblia_acf.csv"; 

// Função de Delay para o Indexador (Crucial para transclusão funcionar na hora)
const sleep = (ms) => new Promise(r => setTimeout(r, ms));

// 2. INPUT
const input = await tp.system.prompt("Referência (ex: gn 5 1,3,5 ou gn 1 1-3)");
if (!input) return;

const partes = input.toLowerCase().trim().split(/\s+/);
let abrev, cap, textoVersiculos;

// Lógica para lidar com "1 sm", "1 pe" etc. vs "gn", "ex"
if (!isNaN(partes[0]) && partes.length > 3) {
    abrev = partes[0] + partes[1];
    cap = partes[2];
    textoVersiculos = partes[3];
} else {
    abrev = partes[0];
    cap = partes[1];
    textoVersiculos = partes[2];
}

const nomeLivro = nomesLivros[abrev];
if (!nomeLivro) {
    new Notice("Livro não reconhecido!");
    return;
}

// --- LÓGICA DE PASTA DINÂMICA (A CORREÇÃO) ---
let pastaDoLivro = "";
const folderBiblia = app.vault.getAbstractFileByPath(pastaRaiz);

if (folderBiblia && folderBiblia.children) {
    // Procura pasta que contenha o nome (ex: "01 Gênesis" contém "Gênesis")
    const pastaEncontrada = folderBiblia.children.find(folder => 
        folder.name.toLowerCase().includes(nomeLivro.toLowerCase())
    );
    pastaDoLivro = pastaEncontrada ? pastaEncontrada.path : `${pastaRaiz}/${nomeLivro}`;
} else {
    pastaDoLivro = `${pastaRaiz}/${nomeLivro}`;
}
// ----------------------------------------------

const nomeNotaCapitulo = `${nomeLivro} ${cap}`;
const caminhoNotaFinal = `${pastaDoLivro}/${nomeNotaCapitulo}.md`;

// 3. CRIAÇÃO DA NOTA (Background)
// Verifica se a nota JÁ existe. Se não existir, cria a partir do CSV.
if (!app.vault.getAbstractFileByPath(caminhoNotaFinal)) {
    
    // Garante que as pastas existem
    if (!app.vault.getAbstractFileByPath(pastaRaiz)) await app.vault.createFolder(pastaRaiz);
    if (!app.vault.getAbstractFileByPath(pastaDoLivro)) await app.vault.createFolder(pastaDoLivro);

    try {
        const fileCSV = app.vault.getAbstractFileByPath(arquivoCSV);
        if (!fileCSV) {
            new Notice("CSV não encontrado na raiz!");
            return;
        }

        const conteudoCSV = await app.vault.read(fileCSV);
        const buscaPrefixo = `${abrev}|${cap}|`;
        const linhas = conteudoCSV.split("\n");
        const versiculosEncontrados = linhas.filter(l => l.startsWith(buscaPrefixo));

        if (versiculosEncontrados.length > 0) {
            let corpoCapitulo = "";
            versiculosEncontrados.forEach(linha => {
                const colunas = linha.split("|");
                // Aplica formato: **1** Texto ^v1
                if(colunas.length >= 4) {
                    corpoCapitulo += `**${colunas[2]}** ${colunas[3].trim()} ^v${colunas[2]}\n\n`;
                }
            });

            // Template atualizado com Tags e Links corretos
            const templateNota = `---
Livro: "[[Fontes/Livros/${nomeLivro}|${nomeLivro}]]"
tags: [biblia]
---

# ${nomeNotaCapitulo}

${corpoCapitulo}`;

            await app.vault.create(caminhoNotaFinal, templateNota);
            new Notice(`Criando e Indexando ${nomeNotaCapitulo}...`);
            
            // Aguarda o Obsidian indexar os blocos ^v para poder citar abaixo
            await sleep(1000); 
            
        } else {
            new Notice("Capítulo não encontrado no CSV.");
            return;
        }
    } catch (e) {
        console.error(e);
        new Notice("Erro: " + e.message);
        return;
    }
}

// 4. PROCESSAMENTO DA LISTA DE VERSÍCULOS PARA O EMBED
let listaVersiculos = [];
if (textoVersiculos) {
    if (textoVersiculos.includes(",")) {
        listaVersiculos = textoVersiculos.split(",").map(v => v.trim()).filter(v => v).map(Number);
    } else if (textoVersiculos.includes("-")) {
        const [inicio, fim] = textoVersiculos.split("-").map(Number);
        for (let i = inicio; i <= fim; i++) { listaVersiculos.push(i); }
    } else {
        listaVersiculos.push(Number(textoVersiculos));
    }
}

// 5. GERAÇÃO DO OUTPUT (Callout com Transclusão)
let output = `> [!Bible] ${nomeLivro} ${cap}:${textoVersiculos}\n`;

listaVersiculos.forEach(v => {
    // Gera ![[Gênesis 1#^v1]]
    output += `> ![[${nomeNotaCapitulo}#^v${v}]]\n`;
});

// Remove o ultimo \n extra se desejar
output += "\n";

tR += output;
%>