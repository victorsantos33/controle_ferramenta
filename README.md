# controle_ferramenta
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<title>Controle de Ferramentas</title>
<style>
  body { font-family: Arial, sans-serif; margin: 20px; }
  label { display: block; margin-top: 10px; }
  input, select { width: 300px; padding: 5px; }
  select[multiple] { height: 100px; }
  button { margin-top: 15px; padding: 8px 12px; cursor: pointer; }
  table { margin-top: 20px; border-collapse: collapse; width: 90%; }
  th, td { border: 1px solid #aaa; padding: 8px; text-align: left; }
  .qtde-container { margin-left: 20px; }
  #relatorioBtn { margin-top: 20px; background: #4CAF50; color: #fff; border: none; border-radius: 5px; }
</style>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
</head>
<body>

<h1>Controle de Ferramentas</h1>

<div id="cadastro">
  <h2>Cadastrar Ferramenta</h2>
  <label>Código: <input type="text" id="codigoFerr" /></label>
  <label>Descrição: <input type="text" id="descFerr" /></label>
  <button onclick="cadastrarFerramenta()">Cadastrar Ferramenta</button>

  <h2>Cadastrar Pessoa</h2>
  <label>Nome: <input type="text" id="nomePessoa" /></label>
  <button onclick="cadastrarPessoa()">Cadastrar Pessoa</button>
</div>

<hr />

<div id="movimentacoes">
  <h2>Registrar Retirada</h2>
  <label>Ferramentas (escolha 1 ou mais):
    <select id="selectFerramenta" multiple onchange="atualizarQtdeFerramentas()"></select>
  </label>
  <div id="qtdeFerramentas" class="qtde-container"></div>

  <label>Pessoas (múltiplas):
    <select id="selectPessoa" multiple></select>
  </label>
  <label>Data Prevista Devolução: <input type="date" id="dataPrevDevol" /></label>
  <button onclick="registrarRetirada()">Registrar Retirada</button>

  <h2>Registrar Devolução</h2>
  <label>Movimentação:
    <select id="selectMovRetirada"></select>
  </label>
  <label>Quantidade Devolvida: <input type="number" id="qtdeDevolvida" min="1" /></label>
  <label>Conferido por:
    <select id="selectPessoaDevol"></select>
  </label>
  <label>Condição:
    <select id="condicaoDevolucao">
      <option value="Bom">Bom</option>
      <option value="Danificado">Danificado</option>
      <option value="Perdido">Perdido</option>
    </select>
  </label>
  <button onclick="registrarDevolucao()">Registrar Devolução</button>
</div>

<hr />

<h2>Histórico</h2>
<button id="relatorioBtn" onclick="gerarRelatorioGeral()">Gerar Relatório Geral (PDF)</button>

<table id="tabelaHistorico">
  <thead>
    <tr>
      <th>ID Mov</th><th>Ferramentas</th><th>Retirado Por</th><th>Data Retirada</th><th>Status</th><th>Qtde Devolvida</th><th>Condição</th><th>Data Devolução</th><th>Relatório</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
let ferramentas = JSON.parse(localStorage.getItem('ferramentas')) || [];
let pessoas = JSON.parse(localStorage.getItem('pessoas')) || [];
let movimentacoes = JSON.parse(localStorage.getItem('movimentacoes')) || [];

function salvarDados() {
  localStorage.setItem('ferramentas', JSON.stringify(ferramentas));
  localStorage.setItem('pessoas', JSON.stringify(pessoas));
  localStorage.setItem('movimentacoes', JSON.stringify(movimentacoes));
}

function cadastrarFerramenta() {
  const codigo = document.getElementById('codigoFerr').value.trim();
  const descricao = document.getElementById('descFerr').value.trim();
  if(!codigo || !descricao) {
    alert("Preencha código e descrição.");
    return;
  }
  const id = Date.now().toString();
  ferramentas.push({id, codigo, descricao});
  salvarDados();
  alert("Ferramenta cadastrada!");
  document.getElementById('codigoFerr').value = '';
  document.getElementById('descFerr').value = '';
  atualizarSelects();
}

function cadastrarPessoa() {
  const nome = document.getElementById('nomePessoa').value.trim();
  if(!nome) {
    alert("Preencha o nome.");
    return;
  }
  const id = Date.now().toString();
  pessoas.push({id, nome});
  salvarDados();
  alert("Pessoa cadastrada!");
  document.getElementById('nomePessoa').value = '';
  atualizarSelects();
}

function atualizarSelects() {
  const selFerr = document.getElementById('selectFerramenta');
  const selPessoa = document.getElementById('selectPessoa');
  const selPessoaDevol = document.getElementById('selectPessoaDevol');
  const selMovRet = document.getElementById('selectMovRetirada');

  [selFerr, selPessoa, selPessoaDevol].forEach(sel => {
    while(sel.firstChild) sel.removeChild(sel.firstChild);
  });
  
  ferramentas.forEach(f => {
    const opt = document.createElement('option');
    opt.value = f.id;
    opt.textContent = f.codigo + " - " + f.descricao;
    selFerr.appendChild(opt);
  });
  
  pessoas.forEach(p => {
    [selPessoa, selPessoaDevol].forEach(sel => {
      const opt = document.createElement('option');
      opt.value = p.id;
      opt.textContent = p.nome;
      sel.appendChild(opt);
    });
  });

  while(selMovRet.firstChild) selMovRet.removeChild(selMovRet.firstChild);
  movimentacoes.filter(m => m.status === "Retirada").forEach(m => {
    const opt = document.createElement('option');
    opt.value = m.idMov;
    opt.textContent = `#${m.idMov} - ${m.retiradoPor} (${m.ferramentas.length} itens)`;
    selMovRet.appendChild(opt);
  });
}

function atualizarQtdeFerramentas() {
  const ferrSelecionadas = Array.from(document.getElementById('selectFerramenta').selectedOptions).map(opt => opt.value);
  const container = document.getElementById('qtdeFerramentas');
  container.innerHTML = '';
  ferrSelecionadas.forEach(id => {
    const f = ferramentas.find(fe => fe.id === id);
    const div = document.createElement('div');
    div.innerHTML = `${f.codigo} - ${f.descricao}: <input type="number" min="1" value="1" data-id="${id}" />`;
    container.appendChild(div);
  });
}

function registrarRetirada() {
  const ferrSelecionadas = Array.from(document.getElementById('selectFerramenta').selectedOptions).map(opt => opt.value);
  const pessoasSelecionadas = Array.from(document.getElementById('selectPessoa').selectedOptions).map(opt => opt.value);
  const dataPrev = document.getElementById('dataPrevDevol').value;

  if (ferrSelecionadas.length < 1) {
    alert("Selecione pelo menos uma ferramenta.");
    return;
  }
  if (pessoasSelecionadas.length === 0 || !dataPrev) {
    alert("Selecione pelo menos uma pessoa e a data prevista de devolução.");
    return;
  }

  const qtdeInputs = document.querySelectorAll('#qtdeFerramentas input');
  const ferramentasComQtde = Array.from(qtdeInputs).map(inp => ({
    id: inp.getAttribute('data-id'),
    quantidade: parseInt(inp.value) || 1
  }));

  const idMov = movimentacoes.length ? movimentacoes[movimentacoes.length-1].idMov + 1 : 1;
  const nomesPessoas = pessoasSelecionadas.map(id => {
    const p = pessoas.find(pe => pe.id === id);
    return p ? p.nome : '';
  }).join(", ");

  movimentacoes.push({
    idMov,
    dataHoraRetirada: new Date().toLocaleString(),
    ferramentas: ferramentasComQtde,
    retiradoPor: nomesPessoas,
    prevDevolucao: dataPrev,
    dataHoraDevolucao: '',
    qtdeDevolvida: 0,
    conferidoPor: '',
    condicaoDevolucao: '',
    status: "Retirada",
    obs: ''
  });
  salvarDados();
  alert("Retirada registrada.");
  atualizarSelects();
  carregarHistorico();
}

function registrarDevolucao() {
  const idMov = parseInt(document.getElementById('selectMovRetirada').value);
  const qtdeDev = parseInt(document.getElementById('qtdeDevolvida').value);
  const idPessoa = document.getElementById('selectPessoaDevol').value;
  const condicao = document.getElementById('condicaoDevolucao').value;
  if(!idMov || !qtdeDev || !idPessoa || !condicao) {
    alert("Preencha todos os campos.");
    return;
  }
  const mov = movimentacoes.find(m => m.idMov === idMov);
  const pessoa = pessoas.find(p => p.id === idPessoa);
  if(!mov) {
    alert("Movimentação não encontrada.");
    return;
  }
  mov.qtdeDevolvida = qtdeDev;
  mov.conferidoPor = pessoa ? pessoa.nome : '';
  mov.condicaoDevolucao = condicao;
  mov.dataHoraDevolucao = new Date().toLocaleString();
  mov.status = "Devolvido";
  salvarDados();
  alert("Devolução registrada.");
  atualizarSelects();
  carregarHistorico();
}

function gerarPDF(mov) {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();

  doc.setFontSize(16);
  doc.text("Relatório de Retirada de Ferramentas", 10, 15);

  doc.setFontSize(12);
  doc.text(`ID Movimentação: ${mov.idMov}`, 10, 30);
  doc.text(`Data Retirada: ${mov.dataHoraRetirada}`, 10, 40);
  doc.text(`Prevista Devolução: ${mov.prevDevolucao}`, 10, 50);
  doc.text(`Retirado Por: ${mov.retiradoPor}`, 10, 60);

  doc.text("Ferramentas:", 10, 75);
  let y = 85;
  mov.ferramentas.forEach(fq => {
    const f = ferramentas.find(fe => fe.id === fq.id);
    doc.text(`- ${f ? f.codigo + " - " + f.descricao : 'Desconhecido'} | Qtde: ${fq.quantidade}`, 10, y);
    y += 10;
  });

  doc.save(`Retirada_${mov.idMov}.pdf`);
}

function gerarRelatorioGeral() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  doc.setFontSize(16);
  doc.text("Relatório Geral de Movimentações", 10, 15);

  let y = 30;
  movimentacoes.forEach(m => {
    doc.setFontSize(12);
    doc.text(`ID: ${m.idMov} | Status: ${m.status}`, 10, y);
    y += 7;
    doc.text(`Retirado Por: ${m.retiradoPor}`, 10, y);
    y += 7;
    doc.text(`Data Retirada: ${m.dataHoraRetirada}`, 10, y);
    y += 7;
    if (m.dataHoraDevolucao) {
      doc.text(`Devolução: ${m.dataHoraDevolucao} | Condição: ${m.condicaoDevolucao}`, 10, y);
      y += 7;
    }
    doc.text("Ferramentas:", 10, y);
    y += 7;
    m.ferramentas.forEach(fq => {
      const f = ferramentas.find(fe => fe.id === fq.id);
      doc.text(`  - ${f ? f.codigo + " - " + f.descricao : ''} (x${fq.quantidade})`, 10, y);
      y += 7;
    });
    y += 10;
    if (y > 270) {
      doc.addPage();
      y = 20;
    }
  });
  doc.save("Relatorio_Geral.pdf");
}

function carregarHistorico() {
  const tbody = document.getElementById('tabelaHistorico').querySelector('tbody');
  tbody.innerHTML = '';
  movimentacoes.forEach(m => {
    const ferrNomes = m.ferramentas.map(fq => {
      const f = ferramentas.find(fe => fe.id === fq.id);
      return f ? `${f.codigo} - ${f.descricao} (x${fq.quantidade})` : '';
    }).join("<br>");
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${m.idMov}</td>
      <td>${ferrNomes}</td>
      <td>${m.retiradoPor}</td>
      <td>${m.dataHoraRetirada}</td>
      <td>${m.status}</td>
      <td>${m.qtdeDevolvida || ''}</td>
      <td>${m.condicaoDevolucao || ''}</td>
      <td>${m.dataHoraDevolucao || ''}</td>
      <td><button onclick='gerarPDF(${JSON.stringify(m)})'>PDF</button></td>
    `;
    tbody.appendChild(tr);
  });
}

atualizarSelects();
carregarHistorico();
</script>

</body>
</html>
