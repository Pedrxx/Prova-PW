/* README */

















































/* README */
npx react-crete-app nome-projeto

npm install 

npm install axios 

npm install react-router-dom

npm install styled-components

npm install sockjs-client @stomp/stompjs

npm install primereact primeicons

npm install react-spring


export class ProdutoService{
	URL = "http://localhost:8080/produtos";

	inserir(produto){
		return axios.post(this.URL, produto);
	}

	alterar(produto){
		return axios.put(this.URL, produto);
	}

	excluir(id){
		return axios.delete(this.URL+"/"+id);
	}

	listar(page, size){
		return axios.get(this.URL+"?page="+page+"&size="+size);
	}
}


import React, { useEffect, useState } from "react";
import { useNavigate, useLocation, useParams } from 'react-router-dom';
import './ProdutoFormulario.css';
import { ProdutoService } from "../../../services/ProdutoService";

const ProdutoFormulario = (props) => {
	const navigate = useNavigate();
	const produtoNovo = { descricao: '', valor: 0, valorPromocional: 0 };
	const location = useLocation();
	const { produtoAlterar } = location.state || {};

	const [produto, setProduto] = useState(produtoNovo);
	const produtoService = new ProdutoService();

	useEffect(() => {
		if(produtoAlterar){
			setProduto(produtoAlterar);
		}else{
			setProduto(produtoNovo);
		}		
	}, []);

	const listaProdutos = () =>{
		navigate("/produtos")
	}

	const alterarValor = (event) => {
		setProduto({ ...produto, [event.target.name]: event.target.value });
	}

	const salvar = () => {
		if (produto.id) {
			produtoService.alterar(produto).then(data => {
				console.log(data);
				setProduto(produtoNovo);
			});
		} else {
			produtoService.inserir(produto).then(data => {
				console.log(data);
				setProduto(produtoNovo);
			});
		}
	}

	return (
		<div style={{ padding: '10px' }}>
			<h2>Inserir ou Alterar um Produto</h2>
			<input type="text" name="descricao" value={produto.descricao} onChange={alterarValor} /><br /><br />
			<input type="number" name="valor" value={produto.valor} onChange={alterarValor} /><br /><br />
			<input type="number" name="valorPromocional" value={produto.valorPromocional} onChange={alterarValor} /><br /><br />
			<button onClick={salvar}>Salvar</button>
			<button onClick={listaProdutos}>Lista Produtos</button>
		</div>
	);
}

export default ProdutoFormulario;


import React, { useEffect, useState } from "react";
import "./ProdutoLista.css";
import { useNavigate } from "react-router-dom";
import { ProdutoService } from "../../../services/ProdutoService";
import { DataTable } from "primereact/datatable";
import { Column } from "primereact/column";
import { Button } from "primereact/button";
import { ConfirmDialog } from "primereact/confirmdialog";
import { Paginator } from "primereact/paginator";

const ProdutoLista = () => {
  const navigate = useNavigate();
  const [produtos, setProdutos] = useState([]);
  const produtoService = new ProdutoService();
  const [idExcluir, setIdExcluir] = useState(null);
  const [dialogExcluir, setDialogExcluir] = useState(false);
  const [first, setFirst] = useState(0);
  const [rows, setRows] = useState(5);

  useEffect(() => {
    buscarProdutos();
  }, [first, rows]);

  const onPageChange = (event) =>{
	setFirst(event.first);
	setRows(event.rows);
  }

  const buscarProdutos = () => {
	const page = first/rows;
    produtoService.listar(page, rows).then((data) => {
      setProdutos(data.data);
    });
  };

  const formulario = () => {
    navigate("/produto-formulario");
  };

  const alterar = (rowData) => {
    //console.log(rowData);
    navigate("/produto-formulario", { state: { produtoAlterar: rowData } });
  };

  const excluir = () => {
    produtoService.excluir(idExcluir).then((data) => {
      buscarProdutos();
    });
  };

  const optionColumn = (rowData) => {
    return (
      <>
        <Button
          label="Alterar"
          severity="warning"
          onClick={() => alterar(rowData)}
        />

        <Button
          label="Excluir"
          severity="danger"
          onClick={() => {
            setIdExcluir(rowData.id);
            setDialogExcluir(true);
          }}
        />
      </>
    );
  };

  return (
    <div className="container">
      <h2>Lista de Produtos</h2>
      <button onClick={formulario}>Novo Produto</button>
      <br />
      <br />
      <DataTable value={produtos.content} tableStyle={{ minWidth: "50rem" }}>
        <Column field="id" header="Id"></Column>
        <Column field="descricao" header="Descrição"></Column>
        <Column field="valor" header="Valor"></Column>
        <Column field="valorPromocional" header="Valor Promocional"></Column>
        <Column header="Opções" body={optionColumn}></Column>
      </DataTable>
      <Paginator
        first={first}
        rows={rows}
        totalRecords={produtos.totalElements}
        rowsPerPageOptions={[5, 10, 20, 30]}
        onPageChange={onPageChange}
      />

      <ConfirmDialog
        visible={dialogExcluir}
        onHide={() => setDialogExcluir(false)}
        message="Deseja excluir?"
        header="Confirmação"
        icon="pi pi-exclamation-triangle"
        accept={excluir}
        reject={() => setIdExcluir(null)}
        acceptLabel="Sim"
        rejectLabel="Não"
      />

      {
        /* 	{produtos.map((produto)=>
				<p key={produto.id}>{produto.descricao} {produto.valor}</p>
			)} */
      }
    </div>
  );
};

export default ProdutoLista;


import React, { useContext } from 'react';
import './Menu.css';
import { TemaContexto } from '../../App';
import { useNavigate } from 'react-router-dom';

const Menu = () => {
	const {dark, setDark} = useContext(TemaContexto);
	const navigate = useNavigate();

	const navegar = (pagina)=>{
		navigate(pagina);
	}
	
  return (
    <div className={`menu ${dark?'dark':'light'}`}>
      <ul>
        <li onClick={() =>navegar("/")}>Home</li>
		    <li onClick={() => navegar("/produtos")}>Produtos</li>
		    <li onClick={() => navegar("/pessoas")}>Pessoas</li>
		    <li onClick={() => navegar("/cidades")}>Cidades</li>
        <li onClick={() =>setDark(!dark)}>Mudar Tema</li>		
      </ul>
    </div>
  );
};

export default Menu;


import React, { createContext, useState } from 'react';
import './App.css';
import Menu from './components/menu/Menu';
import Home from './pages/home/Home';
import Rodape from './components/rodape/Rodape';
import { BrowserRouter, Route, Routes } from 'react-router-dom';
import ProdutoLista from './pages/produto/lista/ProdutoLista';
import ProdutoFormulario from './pages/produto/formulario/ProdutoFormulario';
import PessoaLista from './pages/pessoa/PessoaLista';
import PessoaFormulario from './pages/pessoa/PessoaFormulario';
import CidadeFormulario from './pages/cidade/CidadeFormulario';
import CidadeLista from './pages/cidade/CidadeLista';

export const TemaContexto = createContext();


function App() {
  const [dark, setDark] = useState(true);

  return (
    <div className="App">
      <TemaContexto.Provider value={{ dark, setDark }}>
        <BrowserRouter>
          <Menu />
          <Routes>
            <Route exact path='/' element={<Home />} />
            <Route path='/produtos' element={<ProdutoLista />} />
            <Route path='/produto-formulario' element={<ProdutoFormulario />} />
            <Route path='/pessoas' element={<PessoaLista />} />
            <Route path='/pessoa-formulario' element={<PessoaFormulario />} />
            <Route path='/cidades' element={<CidadeLista />} />
            <Route path='/cidade-formulario' element={<CidadeFormulario />} />
          </Routes>
          <Rodape />
        </BrowserRouter>
      </TemaContexto.Provider>
    </div>
  );
}

export default App;