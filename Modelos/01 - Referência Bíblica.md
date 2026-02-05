
<%*
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
// VERIFIQUE SE O CSV ESTÁ AQUI OU NA RAIZ (se estiver na raiz, remova o 'Notas/Bíblia/')
const arquivoCSV = "biblia_acf.csv"; 

// 2. PROCESSAMENTO DO INPUT
const refInput = await tp.system.prompt("Referência Limpa (ex: gn 4 3-5 ou gn 4 1,3,7)");
if (!refInput) return;

const partes = refInput.toLowerCase().trim().split(/\s+/);
let abrev, cap, textoVersiculos;

// Lógica para livros numerados e detecção de versículos
if (!isNaN(partes[0]) && partes.length > 2) { 
    abrev = partes[0] + partes[1];
    cap = partes[2];
    textoVersiculos = partes[3];
} else {
    abrev = partes[0];
    cap = partes[1];
    textoVersiculos = partes[2];
}

const nomeLivroBase = nomesLivros[abrev];
if (!nomeLivroBase) {
    new Notice("Livro não reconhecido.");
    return;
}

// --- CORREÇÃO PRINCIPAL: LOCALIZAR A PASTA CORRETA COM O NÚMERO ---
let pastaDoLivro = "";
const folderBiblia = app.vault.getAbstractFileByPath(pastaRaiz);

if (folderBiblia && folderBiblia.children) {
    // Procura uma pasta que contenha o nome do livro (ex: "01 Gênesis" contém "Gênesis")
    const pastaEncontrada = folderBiblia.children.find(folder => 
        folder.name.toLowerCase().includes(nomeLivroBase.toLowerCase())
    );
    
    if (pastaEncontrada) {
        pastaDoLivro = pastaEncontrada.path;
    } else {
        // Se não achar (caso raro se rodou o script anterior), define o padrão sem número
        pastaDoLivro = `${pastaRaiz}/${nomeLivroBase}`;
    }
} else {
    // Fallback se a pasta raiz não existir
    pastaDoLivro = `${pastaRaiz}/${nomeLivroBase}`;
}
// ------------------------------------------------------------------

const nomeNotaCapitulo = `${nomeLivroBase} ${cap}`;
const caminhoNotaFinal = `${pastaDoLivro}/${nomeNotaCapitulo}.md`;

// 3. GARANTIR CRIAÇÃO DO CAPÍTULO (Lazy Loading)
const garantirPasta = async (caminho) => {
    if (!app.vault.getAbstractFileByPath(caminho)) {
        await app.vault.createFolder(caminho);
    }
};

if (!app.vault.getAbstractFileByPath(caminhoNotaFinal)) {
    // Apenas tenta criar se a nota NÃO existir
    try {
        await garantirPasta(pastaRaiz);
        await garantirPasta(pastaDoLivro);

        const arquivoCSVFile = app.vault.getAbstractFileByPath(arquivoCSV);
        if (!arquivoCSVFile) throw new Error("Arquivo CSV não encontrado: " + arquivoCSV);

        const conteudoCSV = await app.vault.read(arquivoCSVFile);
        const linhas = conteudoCSV.split("\n");
        
        // Ajuste: O CSV original usa pipes. Certifique-se que abrev bate com o CSV.
        const buscaPrefixo = `${abrev}|${cap}|`; 
        const versiculosEncontrados = linhas.filter(l => l.startsWith(buscaPrefixo));

        if (versiculosEncontrados.length > 0) {
            let corpoCapitulo = "";
            versiculosEncontrados.forEach(linha => {
                const colunas = linha.split("|");
                if (colunas.length >= 4) {
                    // Adiciona negrito e quebra de linha extra
                    corpoCapitulo += `**${colunas[2]}** ${colunas[3].trim()} ^v${colunas[2]}\n\n`;
                }
            });

            // Adicionei tags: [biblia] conforme seu padrão
            const templateNota = `---\ntipo: capítulo\nLivro: "[[Fontes/Livros/${nomeLivroBase}|${nomeLivroBase}]]"\ntags: [biblia]\n---\n\n# ${nomeNotaCapitulo}\n\n${corpoCapitulo}`;
            
            await app.vault.create(caminhoNotaFinal, templateNota);
            new Notice(`${nomeNotaCapitulo} criado com sucesso.`);
        } else {
             new Notice(`Nenhum versículo encontrado para ${abrev} ${cap} no CSV.`);
        }
    } catch (e) {
        console.error(e);
        new Notice("Erro ao gerar nota: " + e.message);
    }
}

// 4. LÓGICA DO LINK E DESTAQUE (HIGHLIGHT)
let primeiroVersiculo;
if (textoVersiculos) {
    if (textoVersiculos.includes(",")) {
        primeiroVersiculo = textoVersiculos.split(",")[0].trim();
    } else if (textoVersiculos.includes("-")) {
        primeiroVersiculo = textoVersiculos.split("-")[0].trim();
    } else {
        primeiroVersiculo = textoVersiculos;
    }
}

// 5. INSERÇÃO DO LINK FORMATADO
// Usa nomeNotaCapitulo que é limpo (Ex: Gênesis 1), o caminho completo o Obsidian resolve
let linkFinal = `[[${nomeNotaCapitulo}`;
let textoDisplay = `${nomeLivroBase} ${cap}`;

if (textoVersiculos) {
    linkFinal += `#^v${primeiroVersiculo}`;
    textoDisplay += `.${textoVersiculos}`;
}
linkFinal += `|${textoDisplay}]]`;

tR += linkFinal;
%>