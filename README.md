<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Analisador de Leilão Simples</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body { font-family: Arial, sans-serif; background: #f7f7f7; padding: 20px; }
    .container { max-width: 400px; margin: auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px #ccc; }
    input, button { width: 100%; padding: 10px; margin: 10px 0; }
  </style>
</head>
<body>
  <div class="container">
    <h2>Analisador de Lote</h2>
    <label for="modelo">Modelo (ex: Biz 110i)</label>
    <input id="modelo" type="text" placeholder="Digite o modelo" />
    
    <label for="lance">Lance Atual (R$)</label>
    <input id="lance" type="number" step="0.01" placeholder="Digite o lance" />
    
    <label for="custos">Gastos Totais (reparo, doc, transporte)</label>
    <input id="custos" type="number" step="0.01" placeholder="Digite os custos" />
    
    <button id="btnAnalisar">Analisar</button>
    
    <div id="resultado" style="margin-top: 20px;"></div>
  </div>

  <script>
    async function consultarFipe(modelo) {
      try {
        // Busca marcas moto (foco em Honda)
        const marcas = await fetch('https://parallelum.com.br/fipe/api/v1/motos/marcas').then(r => r.json());
        const honda = marcas.find(m => m.nome.toLowerCase().includes('honda'));
        if(!honda) return null;
        
        // Busca modelos da Honda
        const modelos = await fetch(`https://parallelum.com.br/fipe/api/v1/motos/marcas/${honda.codigo}/modelos`).then(r => r.json());
        const moto = modelos.modelos.find(m => m.nome.toLowerCase().includes(modelo.toLowerCase()));
        if(!moto) return null;
        
        // Busca anos disponíveis
        const anos = await fetch(`https://parallelum.com.br/fipe/api/v1/motos/marcas/${honda.codigo}/modelos/${moto.codigo}/anos`).then(r => r.json());
        if(anos.length === 0) return null;
        
        // Busca preço FIPE do ano mais recente
        const preco = await fetch(`https://parallelum.com.br/fipe/api/v1/motos/marcas/${honda.codigo}/modelos/${moto.codigo}/anos/${anos[0].codigo}`).then(r => r.json());
        return parseFloat(preco.Valor.replace('R$', '').replace('.', '').replace(',', '.'));
      } catch {
        return null;
      }
    }

    document.getElementById('btnAnalisar').addEventListener('click', async () => {
      const modelo = document.getElementById('modelo').value.trim();
      const lance = parseFloat(document.getElementById('lance').value);
      const custos = parseFloat(document.getElementById('custos').value);
      const resultadoDiv = document.getElementById('resultado');
      
      if(!modelo || isNaN(lance) || isNaN(custos)) {
        resultadoDiv.innerHTML = '<p style="color:red;">Preencha todos os campos corretamente.</p>';
        return;
      }

      resultadoDiv.innerHTML = 'Consultando FIPE...';

      const fipe = await consultarFipe(modelo);
      if(fipe === null) {
        resultadoDiv.innerHTML = '<p style="color:red;">Modelo não encontrado na FIPE.</p>';
        return;
      }

      const custoTotal = lance + custos;
      const lucro = fipe - custoTotal;

      let classificacao = '';
      if(lucro > 2000) classificacao = '✅ Alta Oportunidade';
      else if(lucro > 1000) classificacao = '⚠️ Oportunidade Razoável';
      else if(lucro > 0) classificacao = '❗ Margem apertada';
      else classificacao = '❌ Prejuízo ou não compensa';

      resultadoDiv.innerHTML = `
        <p><b>Valor FIPE:</b> R$ ${fipe.toFixed(2)}</p>
        <p><b>Custo Total:</b> R$ ${custoTotal.toFixed(2)}</p>
        <p><b>Lucro Estimado:</b> R$ ${lucro.toFixed(2)}</p>
        <p><b>Classificação:</b> ${classificacao}</p>
      `;
    });
  </script>
</body>
</html>
