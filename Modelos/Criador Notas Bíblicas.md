<%*
// =============================================================================
// SCRIPT DE GERAÇÃO DA BÍBLIA PARA OBSIDIAN (V2 - Melhorado)
// =============================================================================

// 1. Configurações e Mapeamento
// -----------------------------------------------------------------------------
const csvFileName = "biblia_acf.csv"; // O arquivo deve estar na raiz do vault
const baseFolderNotas = "Notas/Bíblia";
const baseFolderLivros = "Fontes/Livros";
const baseFolderBios = "Fontes/Biografias";

// Mapeamento manual de Autor e Ordem
const bookMap = {
    'gn': {order: 1, name: 'Gênesis', author: 'Moisés'},
    'ex': {order: 2, name: 'Êxodo', author: 'Moisés'},
    'lv': {order: 3, name: 'Levítico', author: 'Moisés'},
    'nm': {order: 4, name: 'Números', author: 'Moisés'},
    'dt': {order: 5, name: 'Deuteronômio', author: 'Moisés'},
    'js': {order: 6, name: 'Josué', author: 'Josué'},
    'jz': {order: 7, name: 'Juízes', author: 'Samuel'},
    'rt': {order: 8, name: 'Rute', author: 'Samuel'},
    '1sm': {order: 9, name: '1 Samuel', author: 'Samuel'},
    '2sm': {order: 10, name: '2 Samuel', author: 'Samuel'},
    '1rs': {order: 11, name: '1 Reis', author: 'Jeremias'},
    '2rs': {order: 12, name: '2 Reis', author: 'Jeremias'},
    '1cr': {order: 13, name: '1 Crônicas', author: 'Esdras'},
    '2cr': {order: 14, name: '2 Crônicas', author: 'Esdras'},
    'ed': {order: 15, name: 'Esdras', author: 'Esdras'},
    'ne': {order: 16, name: 'Neemias', author: 'Neemias'},
    'et': {order: 17, name: 'Ester', author: 'Mardoqueu'},
    'jó': {order: 18, name: 'Jó', author: 'Moisés'},
    'sl': {order: 19, name: 'Salmos', author: 'Davi'},
    'pv': {order: 20, name: 'Provérbios', author: 'Salomão'},
    'ec': {order: 21, name: 'Eclesiastes', author: 'Salomão'},
    'ct': {order: 22, name: 'Cânticos', author: 'Salomão'},
    'is': {order: 23, name: 'Isaías', author: 'Isaías'},
    'jr': {order: 24, name: 'Jeremias', author: 'Jeremias'},
    'lm': {order: 25, 'name': 'Lamentações', author: 'Jeremias'},
    'ez': {order: 26, name: 'Ezequiel', author: 'Ezequiel'},
    'dn': {order: 27, name: 'Daniel', author: 'Daniel'},
    'os': {order: 28, 'name': 'Oséias', author: 'Oséias'},
    'jl': {order: 29, name: 'Joel', author: 'Joel'},
    'am': {order: 30, 'name': 'Amós', author: 'Amós'},
    'ob': {order: 31, 'name': 'Obadias', author: 'Obadias'},
    'jn': {order: 32, 'name': 'Jonas', author: 'Jonas'},
    'mq': {order: 33, 'name': 'Miquéias', author: 'Miquéias'},
    'na': {order: 34, 'name': 'Naum', author: 'Naum'},
    'hc': {order: 35, 'name': 'Habacuque', author: 'Habacuque'},
    'sf': {order: 36, 'name': 'Sofonias', author: 'Sofonias'},
    'ag': {order: 37, 'name': 'Ageu', author: 'Ageu'},
    'zc': {order: 38, 'name': 'Zacarias', author: 'Zacarias'},
    'ml': {order: 39, 'name': 'Malaquias', author: 'Malaquias'},
    'mt': {order: 40, 'name': 'Mateus', author: 'Mateus'},
    'mc': {order: 41, 'name': 'Marcos', author: 'Marcos'},
    'lc': {order: 42, 'name': 'Lucas', author: 'Lucas'},
    'jo': {order: 43, 'name': 'João', author: 'João'},
    'at': {order: 44, 'name': 'Atos', author: 'Lucas'},
    'rm': {order: 45, 'name': 'Romanos', author: 'Paulo'},
    '1co': {order: 46, 'name': '1 Coríntios', author: 'Paulo'},
    '2co': {order: 47, 'name': '2 Coríntios', author: 'Paulo'},
    'gl': {order: 48, 'name': 'Gálatas', author: 'Paulo'},
    'ef': {order: 49, 'name': 'Efésios', author: 'Paulo'},
    'fp': {order: 50, 'name': 'Filipenses', author: 'Paulo'},
    'cl': {order: 51, 'name': 'Colossenses', author: 'Paulo'},
    '1ts': {order: 52, 'name': '1 Tessalonicenses', author: 'Paulo'},
    '2ts': {order: 53, 'name': '2 Tessalonicenses', author: 'Paulo'},
    '1tm': {order: 54, 'name': '1 Timóteo', 'author': 'Paulo'},
    '2tm': {order: 55, 'name': '2 Timóteo', 'author': 'Paulo'},
    'tt': {order: 56, 'name': 'Tito', 'author': 'Paulo'},
    'fm': {order: 57, 'name': 'Filemom', 'author': 'Paulo'},
    'hb': {order: 58, 'name': 'Hebreus', author: 'Desconhecido'},
    'tg': {order: 59, 'name': 'Tiago', author: 'Tiago'},
    '1pe': {order: 60, 'name': '1 Pedro', author: 'Pedro'},
    '2pe': {order: 61, 'name': '2 Pedro', author: 'Pedro'},
    '1jo': {order: 62, 'name': '1 João', author: 'João'},
    '2jo': {order: 63, 'name': '2 João', author: 'João'},
    '3jo': {order: 64, 'name': '3 João', author: 'João'},
    'jd': {order: 65, 'name': 'Judas', author: 'Judas'},
    'ap': {order: 66, 'name': 'Apocalipse', author: 'João'}
};

// Funções Auxiliares
async function createFolderIfNotExists(path) {
    if (!app.vault.getAbstractFileByPath(path)) {
        await app.vault.createFolder(path);
    }
}

async function createFileIfNotExists(path, content) {
    const file = app.vault.getAbstractFileByPath(path);
    if (!file) {
        await app.vault.create(path, content);
    } else {
        // Se quiser sobrescrever, descomente a linha abaixo:
        // await app.vault.modify(file, content);
    }
}

// 2. Leitura e Processamento do CSV
// -----------------------------------------------------------------------------
const file = app.vault.getAbstractFileByPath(csvFileName);
if (!file) {
    new Notice(`ERRO: Arquivo ${csvFileName} não encontrado na raiz do vault!`);
    return;
}

const content = await app.vault.read(file);
const lines = content.split('\n');

const bibleData = {};

lines.forEach(line => {
    if (!line.trim()) return;
    const parts = line.split('|');
    if (parts.length < 4) return;

    const code = parts[0].trim();
    const chapter = parts[1].trim();
    const verse = parts[2].trim();
    const text = parts[3].trim();

    if (!bibleData[code]) bibleData[code] = {};
    if (!bibleData[code][chapter]) bibleData[code][chapter] = [];

    // MUDANÇA 1: Número em negrito no início (**1**)
    bibleData[code][chapter].push(`**${verse}** ${text} ^v${verse}`);
});

// 3. Geração das Notas
// -----------------------------------------------------------------------------
new Notice("Iniciando geração da Bíblia (V2)...");

await createFolderIfNotExists("Notas");
await createFolderIfNotExists("Fontes");
await createFolderIfNotExists(baseFolderNotas);
await createFolderIfNotExists(baseFolderLivros);
await createFolderIfNotExists(baseFolderBios);

const createdAuthors = new Set();
const createdBooks = new Set();

for (const code in bibleData) {
    if (!bookMap[code]) continue;

    const info = bookMap[code];
    const bookName = info.name;
    const authorName = info.author;
    const order = String(info.order).padStart(2, '0');
    
    // --- A. Nota do Autor ---
    if (!createdAuthors.has(authorName)) {
        const authorPath = `${baseFolderBios}/${authorName}.md`;
        const authorContent = 
`---
tags: [biografia, autor_biblico]
---
# ${authorName}

`;
        await createFileIfNotExists(authorPath, authorContent);
        createdAuthors.add(authorName);
    }

    // --- B. Nota do Livro ---
    if (!createdBooks.has(bookName)) {
        const bookPath = `${baseFolderLivros}/${bookName}.md`;
        const bookContent = 
`---
Autor: "[[${baseFolderBios}/${authorName}|${authorName}]]"
tags: [livro_biblico]
---
# ${bookName}

`;
        await createFileIfNotExists(bookPath, bookContent);
        createdBooks.add(bookName);
    }

    // --- C. Notas dos Capítulos ---
    const bookFolder = `${baseFolderNotas}/${order} ${bookName}`;
    await createFolderIfNotExists(bookFolder);

    for (const chapterNum in bibleData[code]) {
        const verses = bibleData[code][chapterNum];
        const fileName = `${bookName} ${chapterNum}.md`;
        const filePath = `${bookFolder}/${fileName}`;
        
        // MUDANÇA 2: Adicionei a propriedade tags: [biblia]
        // MUDANÇA 3: usei .join('\n\n') para dar espaço de uma linha entre versículos
        const chapterContent = 
`---
Livro: "[[${baseFolderLivros}/${bookName}|${bookName}]]"
tags: [biblia]
---

# ${bookName} ${chapterNum}

${verses.join('\n\n')}
`;
        await createFileIfNotExists(filePath, chapterContent);
    }
}

new Notice("Geração da Bíblia V2 Concluída!");
_%>