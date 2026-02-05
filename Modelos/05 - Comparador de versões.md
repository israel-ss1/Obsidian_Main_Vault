<%*
// 1. MAPEAMENTO DE LIVROS
const nomesLivros = {
    "gn": "Gênesis", "ex": "Êxodo", "lv": "Levítico", "nm": "Números", "dt": "Deuteronômio",
    "js": "Josué", "jz": "Juízes", "rt": "Rute", "1sm": "1 Samuel", "2sm": "2 Samuel",
    "1rs": "1 Reis", "2rs": "2 Reis", "1cr": "1 Crônicas", "2cr": "2 Crônicas",
    "ed": "Esdras", "ne": "Neemias", "et": "Ester", "jo": "Jó", "sl": "Salmos",
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

const pastaBiblia = "Notas/Bíblia";
const arquivoACF = "biblia_acf.csv"; 

// 2. SELEÇÃO MÚLTIPLA DE VERSÕES (LOOP)
const folder = app.vault.getAbstractFileByPath(pastaBiblia);
const arquivosCSV = folder.children.filter(f => f.extension === "csv" && f.name !== arquivoACF);

let versoesSelecionadas = [];
let finalizado = false;

while (!finalizado) {
    const nomesRestantes = arquivosCSV
        .filter(f => !versoesSelecionadas.includes(f))
        .map(f => f.name.replace(".csv", "").toUpperCase());
    
    const opcoes = ["✅ FINALIZAR SELEÇÃO", ...nomesRestantes];
    const escolha = await tp.system.suggester(opcoes, opcoes);

    if (!escolha || escolha === "✅ FINALIZAR SELEÇÃO") {
        finalizado = true;
    } else {
        const arquivo = arquivosCSV.find(f => f.name.toUpperCase().includes(escolha));
        versoesSelecionadas.push(arquivo);
        new Notice(`Adicionado: ${escolha}`);
    }
}

if (versoesSelecionadas.length === 0) {
    new Notice("Nenhuma versão extra selecionada. Usando apenas ACF.");
}

// 3. INPUT DA REFERÊNCIA
const input = await tp.system.prompt("Referência (ex: gn 1 1-3)");
if (!input) return;

const partes = input.toLowerCase().trim().split(/\s+/);
let abrev, cap, textoVersiculos;

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
if (!nomeLivro) return;

// 4. CARREGAMENTO DOS DADOS
const fileACF = app.vault.getAbstractFileByPath(`${pastaBiblia}/${arquivoACF}`) || app.vault.getAbstractFileByPath(arquivoACF);
const conteudoACF = await app.vault.read(fileACF);

// 5. PROCESSAR LISTA DE VERSÍCULOS
let listaV = [];
if (textoVersiculos.includes(",")) {
    listaV = textoVersiculos.split(",").map(v => Number(v.trim()));
} else if (textoVersiculos.includes("-")) {
    const [i, f] = textoVersiculos.split("-").map(Number);
    for (let x = i; x <= f; x++) listaV.push(x);
} else {
    listaV.push(Number(textoVersiculos));
}

// 6. GERAÇÃO DO OUTPUT (BOX ÚNICO)
let output = `> [!Bible] Comparação de Versões: ${nomeLivro} ${cap}:${textoVersiculos}\n`;

// SEÇÃO ACF
output += `> **Versão ACF**\n`;
listaV.forEach(v => {
    const busca = `${abrev}|${cap}|${v}|`;
    const linha = conteudoACF.split("\n").find(l => l.startsWith(busca));
    if (linha) output += `> **${v}** ${linha.split("|")[3].trim()}\n>\n`;
});

// SEÇÕES EXTRAS
for (const csv of versoesSelecionadas) {
    output += `> ---\n`;
    const sigla = csv.name.replace(".csv", "").replace("biblia_", "").toUpperCase();
    const conteudoExtra = await app.vault.read(csv);
    
    output += `> **Versão ${sigla}**\n`;
    listaV.forEach(v => {
        const busca = `${abrev}|${cap}|${v}|`;
        const linha = conteudoExtra.split("\n").find(l => l.startsWith(busca));
        if (linha) output += `> **${v}** ${linha.split("|")[3].trim()}\n>\n`;
    });
}

tR += output;

// 7. BACKGROUND: Garantir nota ACF (Lazy Loading)
const nomeNotaCapitulo = `${nomeLivro} ${cap}`;
const pastaDestino = `${pastaBiblia}/${nomeLivro}`;
if (!app.vault.getAbstractFileByPath(`${pastaDestino}/${nomeNotaCapitulo}.md`)) {
    if (!app.vault.getAbstractFileByPath(pastaDestino)) await app.vault.createFolder(pastaDestino);
    const prefixo = `${abrev}|${cap}|`;
    const linhasACF = conteudoACF.split("\n").filter(l => l.startsWith(prefixo));
    let corpo = "";
    linhasACF.forEach(l => {
        const c = l.split("|");
        corpo += `**${c[2]}** ${c[3].trim()} ^v${c[2]}\n\n`;
    });
    const template = `---\ntipo: capítulo\nlivro: "[[${nomeLivro}]]"\n---\n# ${nomeNotaCapitulo}\n\n${corpo}`;
    await app.vault.create(`${pastaDestino}/${nomeNotaCapitulo}.md`, template);
}
%>