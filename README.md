const express = require('express'); const cors = require('cors'); const bcrypt = require('bcrypt'); const jwt = require('jsonwebtoken'); const { v4: uuidv4 } = require('uuid'); const dotenv = require('dotenv');

dotenv.config(); const app = express(); app.use(express.json()); app.use(cors());

const users = []; const comercios = []; const pedidos = [];

// Registro de usuário app.post('/register', async (req, res) => { const { nome, email, senha, tipo } = req.body; const hashedSenha = await bcrypt.hash(senha, 10); const novoUsuario = { id: uuidv4(), nome, email, senha: hashedSenha, tipo }; users.push(novoUsuario); res.status(201).json({ mensagem: 'Usuário registrado com sucesso!' }); });

// Login de usuário app.post('/login', async (req, res) => { const { email, senha } = req.body; const usuario = users.find(u => u.email === email); if (!usuario || !(await bcrypt.compare(senha, usuario.senha))) { return res.status(401).json({ mensagem: 'Credenciais inválidas!' }); } const token = jwt.sign({ id: usuario.id, tipo: usuario.tipo }, process.env.JWT_SECRET, { expiresIn: '1h' }); res.json({ token }); });

// Cadastro de comércio app.post('/comercios', (req, res) => { const { nome, endereco, categoria, dono_id } = req.body; const novoComercio = { id: uuidv4(), nome, endereco, categoria, dono_id }; comercios.push(novoComercio); res.status(201).json({ mensagem: 'Comércio cadastrado com sucesso!' }); });

// Listar comércios app.get('/comercios', (req, res) => { res.json(comercios); });

// Criar pedido app.post('/pedidos', (req, res) => { const { cliente_id, comercio_id, itens, valor_total, endereco_entrega } = req.body; const novoPedido = { id: uuidv4(), cliente_id, comercio_id, itens, valor_total, endereco_entrega, status: 'pendente' }; pedidos.push(novoPedido); res.status(201).json({ mensagem: 'Pedido criado com sucesso!', pedido: novoPedido }); });

// Atualizar status do pedido (Ex: Em preparo, Saiu para entrega, Entregue) app.put('/pedidos/:id', (req, res) => { const { id } = req.params; const { status } = req.body; const pedido = pedidos.find(p => p.id === id); if (!pedido) { return res.status(404).json({ mensagem: 'Pedido não encontrado!' }); } pedido.status = status; res.json({ mensagem: 'Status do pedido atualizado!', pedido }); });

// Listar pedidos por cliente app.get('/pedidos/:cliente_id', (req, res) => { const { cliente_id } = req.params; const pedidosCliente = pedidos.filter(p => p.cliente_id === cliente_id); res.json(pedidosCliente); });

const PORT = process.env.PORT || 3000; app.listen(PORT, () => console.log(Servidor rodando na porta ${PORT}));
