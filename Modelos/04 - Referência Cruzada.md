<%*
// ==========================================================
// 1. DICIONÁRIO DE LIVROS (Igual ao seu script original)
// ==========================================================
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

// ==========================================================
// 2. CALCULAR NÚMERO DO RODAPÉ (1, 2, 3...)
// ==========================================================
const content = tp.file.content;
const regex = /\[\^(\d+)\]/g;
let match;
let max = 0;

// Varre o arquivo para achar o maior número já usado
while ((match = regex.exec(content)) !== null) {
    const num = parseInt(match[1]);
    if (num > max) max = num;
}
const next = max + 1;

// ==========================================================
// 3. INTERFACE COM O USUÁRIO
// ==========================================================
const refInput = await tp.system.prompt("Ref Cruzada (ex: gn 1 1 ou mt 5 3-5)");
if (!refInput) return; // Cancela se der Esc

// ==========================================================
// 4. PROCESSAR A REFERÊNCIA (Lógica do seu script original)
// ==========================================================
const partes = refInput.toLowerCase().trim().split(/\s+/);
let abrev, cap, versiculos;

// Lógica para livros com número (1 sm, 2 pe) vs simples (gn, mt)
if (!isNaN(partes[0]) && partes.length > 2) { 
    abrev = partes[0] + partes[1];
    cap = partes[2];
    versiculos = partes[3];
} else {
    abrev = partes[0];
    cap = partes[1];
    versiculos = partes[2];
}

const nomeLivro = nomesLivros[abrev];
if (!nomeLivro) {
    new Notice("Livro não reconhecido no dicionário.");
    return;
}

// ==========================================================
// 5. CONSTRUÇÃO DO LINK
// ==========================================================
// O nome do arquivo é sempre "Livro Capítulo" (ex: "Gênesis 1")
const nomeNota = `${nomeLivro} ${cap}`;

// Prepara o link. O Obsidian acha o arquivo onde quer que ele esteja 
// desde que o nome seja único.
let linkDestino = `${nomeNota}`;
let textoDisplay = `${nomeLivro} ${cap}`;

// Se tiver versículo, adiciona o anchor #^vNÚMERO
if (versiculos) {
    // Pega apenas o primeiro número se for um intervalo (3-5 -> 3)
    let primeiroVerso = versiculos;
    if (versiculos.includes("-")) primeiroVerso = versiculos.split("-")[0];
    if (versiculos.includes(",")) primeiroVerso = versiculos.split(",")[0];
    
    // IMPORTANTE: Aqui usamos o padrão do seu script criador: ^vNÚMERO
    linkDestino += `#^v${primeiroVerso}`;
    textoDisplay += `:${versiculos}`;
}

// ==========================================================
// 6. INSERÇÃO NO DOCUMENTO
// ==========================================================

// Insere o numerozinho [^1] onde o cursor está
tR += `[^${next}]`;

// Prepara o texto do rodapé: [^1]: [[Gênesis 1#^v1|Gênesis 1:1]]
const rodapeTexto = `\n[^${next}]: [[${linkDestino}|${textoDisplay}]]`;

// Adiciona ao final do arquivo atual
const arquivoAtual = tp.file.find_tfile(tp.file.path(true));
await app.vault.append(arquivoAtual, rodapeTexto);
%>