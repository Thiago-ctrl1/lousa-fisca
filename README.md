# lousa-fisca
importador XML1
import React, { useState } from "react";

export default function ImportadorXML({ onVoltar }) {
  const [total, setTotal] = useState(0);
  const [notas, setNotas] = useState([]);
  const [prestador, setPrestador] = useState("");

  const handleImport = (event) => {
    const arquivos = event.target.files;
    let valores = [];
    let notasTemp = [];

    Array.from(arquivos).forEach((arquivo, index) => {
      const leitor = new FileReader();
      leitor.onload = (e) => {
        const parser = new DOMParser();
        const xml = parser.parseFromString(e.target.result, "text/xml");
        const vServ = xml.getElementsByTagName("vServ");
        let totalNota = 0;
        for (let i = 0; i < vServ.length; i++) {
          const valor = parseFloat(vServ[i].textContent.replace(",", "."));
          if (!isNaN(valor)) {
            valores.push(valor);
            totalNota += valor;
          }
        }

        const numeroNota = xml.getElementsByTagName("nNF")[0]?.textContent || "-";
        const dataEmissao = xml.getElementsByTagName("dhEmi")[0]?.textContent || xml.getElementsByTagName("dEmi")[0]?.textContent || "-";
        const tomador = xml.getElementsByTagName("xNome")[1]?.textContent || "-";
        const prest = xml.getElementsByTagName("xNome")[0]?.textContent || "-";
        if (index === 0) setPrestador(prest);

        notasTemp.push({ numeroNota, dataEmissao, tomador, totalNota });

        if (index === arquivos.length - 1) {
          setNotas(notasTemp);
        }
      };
      leitor.readAsText(arquivo);
    });
  };

  const calcularTotal = () => {
    const soma = notas.reduce((acc, nota) => acc + nota.totalNota, 0);
    setTotal(soma);
  };

  const formatarData = (data) => {
    if (!data) return "-";
    return new Date(data).toLocaleDateString("pt-BR");
  };

  const formatarMoeda = (valor) => {
    return valor.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
  };

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <h2 className="text-xl font-bold">Importador de XML de Serviços</h2>
        <button onClick={onVoltar} className="px-4 py-2 bg-gray-500 text-white rounded">Voltar ao Menu</button>
      </div>

      <div className="flex gap-2">
        <button onClick={() => document.getElementById('inputXML').click()} className="px-4 py-2 bg-blue-600 text-white rounded">Importar XML</button>
        <button onClick={calcularTotal} className="px-4 py-2 bg-green-600 text-white rounded">Calcular</button>
      </div>

      <input
        type="file"
        id="inputXML"
        accept=".xml"
        multiple
        onChange={handleImport}
        className="hidden"
      />

      {prestador && <h3 className="text-lg font-semibold">Prestador: {prestador}</h3>}

      {notas.length > 0 && (
        <table className="w-full table-auto border text-sm">
          <thead className="bg-gray-200">
            <tr>
              <th className="border px-2 py-1">Número</th>
              <th className="border px-2 py-1">Data de Emissão</th>
              <th className="border px-2 py-1">Tomador</th>
              <th className="border px-2 py-1">Valor</th>
            </tr>
          </thead>
          <tbody>
            {notas.map((nota, i) => (
              <tr key={i}>
                <td className="border px-2 py-1">{nota.numeroNota}</td>
                <td className="border px-2 py-1">{formatarData(nota.dataEmissao)}</td>
                <td className="border px-2 py-1">{nota.tomador}</td>
                <td className="border px-2 py-1 text-right">{formatarMoeda(nota.totalNota)}</td>
              </tr>
            ))}
          </tbody>
        </table>
      )}

      <div className="text-right font-semibold">
        Total dos Serviços: {formatarMoeda(total)}
      </div>
    </div>
  );
}
