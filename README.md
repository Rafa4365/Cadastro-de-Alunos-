CREATE TABLE Usuarios (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Nome NVARCHAR(100) NOT NULL,
    Email NVARCHAR(100) NOT NULL,
    Senha NVARCHAR(100) NOT NULL,
    Telefone NVARCHAR(20) NULL
);
ALTER TABLE Usuarios ADD Telefone NVARCHAR(20) NULL;
<%@ Page Language="VB" AutoEventWireup="false" CodeFile="Cadastro.aspx.vb" Inherits="Cadastro" %>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>Cadastro de Usuário</title>
    <style>
        body, div, form { margin:0; padding:0; font-family: Arial, sans-serif; }
        body { background-color:#f4f4f4; display:flex; justify-content:center; align-items:center; height:100vh; }
        .container { background-color:#fff; padding:20px; border-radius:8px; box-shadow:0 2px 10px rgba(0,0,0,0.1); width:300px; text-align:center; }
        h1 { color:#333; }
        form div { margin-bottom:15px; text-align:left; }
        label { display:block; font-size:14px; margin-bottom:5px; color:#666; }
        input { width:100%; padding:8px; font-size:14px; border:1px solid #ccc; border-radius:4px; }
        button { width:100%; padding:10px; background-color:#007bff; color:white; border:none; border-radius:4px; cursor:pointer; }
        button:hover { background-color:#0056b3; }
        #mensagem { margin-top:10px; }
    </style>
</head>
<body>
    <form id="form1" runat="server">
        <div class="container">
            <h1>Cadastro de Usuário</h1>
            <div>
                <label for="txtNome">Nome:</label>
                <input type="text" id="txtNome" runat="server" />
            </div>
            <div>
                <label for="txtEmail">E-mail:</label>
                <input type="email" id="txtEmail" runat="server" />
            </div>
            <div>
                <label for="txtTelefone">Telefone:</label>
                <input type="text" id="txtTelefone" runat="server" maxlength="15" oninput="mascaraTelefone(this)" />
            </div>
            <div>
                <label for="txtSenha">Senha:</label>
                <input type="password" id="txtSenha" runat="server" />
            </div>
            <button id="btnCadastrar" runat="server" onserverclick="btnCadastrar_Click">Cadastrar</button>
            <div id="mensagem" runat="server"></div>
        </div>
    </form>

    <script type="text/javascript">
        function mascaraTelefone(input) {
            let valor = input.value.replace(/\D/g, '');
            if (valor.length > 11) valor = valor.substring(0, 11);
            if (valor.length <= 10) {
                input.value = valor.replace(/(\d{2})(\d{4})(\d{0,4})/, "($1) $2-$3");
            } else {
                input.value = valor.replace(/(\d{2})(\d{5})(\d{0,4})/, "($1) $2-$3");
            }
        }
    </script>
</body>
</html>
Imports System.Data.SqlClient

Partial Class Cadastro
    Inherits System.Web.UI.Page

    Protected Sub btnCadastrar_Click(sender As Object, e As EventArgs)
        Dim nome As String = txtNome.Value.Trim()
        Dim email As String = txtEmail.Value.Trim()
        Dim telefone As String = txtTelefone.Value.Trim()
        Dim senha As String = txtSenha.Value.Trim()

        If String.IsNullOrEmpty(nome) OrElse String.IsNullOrEmpty(email) OrElse String.IsNullOrEmpty(telefone) OrElse String.IsNullOrEmpty(senha) Then
            mensagem.InnerText = "Preencha todos os campos."
            mensagem.Style("color") = "red"
            Return
        End If

        Dim strConexao As String = System.Configuration.ConfigurationManager.ConnectionStrings("ConexaoDB").ConnectionString
        Using conexao As New SqlConnection(strConexao)
            Dim sql As String = "INSERT INTO Usuarios (Nome, Email, Telefone, Senha) VALUES (@Nome, @Email, @Telefone, @Senha)"
            Using cmd As New SqlCommand(sql, conexao)
                cmd.Parameters.AddWithValue("@Nome", nome)
                cmd.Parameters.AddWithValue("@Email", email)
                cmd.Parameters.AddWithValue("@Telefone", telefone)
                cmd.Parameters.AddWithValue("@Senha", senha)
                conexao.Open()
                cmd.ExecuteNonQuery()
            End Using
        End Using

        mensagem.InnerText = "Usuário cadastrado com sucesso!"
        mensagem.Style("color") = "green"

        txtNome.Value = ""
        txtEmail.Value = ""
        txtTelefone.Value = ""
        txtSenha.Value = ""
    End Sub
End Class
<%@ Page Language="VB" AutoEventWireup="false" CodeFile="ListaUsuarios.aspx.vb" Inherits="ListaUsuarios" %>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>Lista de Usuários</title>
    <style>
        body { background:#f4f4f4; font-family: Arial, sans-serif; padding:30px; }
        .tabela { width:100%; border-collapse:collapse; background:#fff; box-shadow:0 2px 10px rgba(0,0,0,0.1); }
        .tabela th, .tabela td { border:1px solid #ddd; padding:8px; text-align:left; }
        .tabela th { background:#007bff; color:#fff; }
        .btn-editar, .btn-excluir { padding:5px 10px; border:none; border-radius:4px; cursor:pointer; }
        .btn-editar { background:#28a745; color:white; }
        .btn-excluir { background:#dc3545; color:white; }
    </style>
</head>
<body>
    <form id="form1" runat="server">
        <h1>Lista de Usuários</h1>
        <asp:GridView ID="GridView1" runat="server" AutoGenerateColumns="False" DataKeyNames="Id"
            CssClass="tabela" OnRowEditing="GridView1_RowEditing"
            OnRowCancelingEdit="GridView1_RowCancelingEdit"
            OnRowUpdating="GridView1_RowUpdating"
            OnRowCommand="GridView1_RowCommand">
            <Columns>
                <asp:BoundField DataField="Id" HeaderText="ID" ReadOnly="True" />
                <asp:BoundField DataField="Nome" HeaderText="Nome" />
                <asp:BoundField DataField="Email" HeaderText="E-mail" />
                <asp:BoundField DataField="Telefone" HeaderText="Telefone" />
                <asp:TemplateField HeaderText="Senha">
                    <ItemTemplate>****</ItemTemplate>
                    <EditItemTemplate>
                        <asp:TextBox ID="txtSenha" runat="server" Text='<%# Bind("Senha") %>' TextMode="Password" />
                    </EditItemTemplate>
                </asp:TemplateField>
                <asp:CommandField ShowEditButton="True" EditText="Editar" CancelText="Cancelar" UpdateText="Salvar"
                    ButtonType="Button" ControlStyle-CssClass="btn-editar" />
                <asp:TemplateField HeaderText="Excluir">
                    <ItemTemplate>
                        <asp:Button ID="btnExcluir" runat="server" Text="Excluir"
                            CommandName="ExcluirUsuario" CommandArgument='<%# Eval("Id") %>'
                            CssClass="btn-excluir"
                            OnClientClick="return confirm('Tem certeza que deseja excluir este usuário?');" />
                    </ItemTemplate>
                </asp:TemplateField>
            </Columns>
        </asp:GridView>
    </form>
</body>
</html>
Imports System.Data.SqlClient

Partial Class ListaUsuarios
    Inherits System.Web.UI.Page

    Protected Sub Page_Load(sender As Object, e As EventArgs) Handles Me.Load
        If Not IsPostBack Then
            CarregarUsuarios()
        End If
    End Sub

    Private Sub CarregarUsuarios()
        Dim strConexao As String = System.Configuration.ConfigurationManager.ConnectionStrings("ConexaoDB").ConnectionString
        Using conexao As New SqlConnection(strConexao)
            Dim sql As String = "SELECT Id, Nome, Email, Telefone, Senha FROM Usuarios ORDER BY Id DESC"
            Using cmd As New SqlCommand(sql, conexao)
                conexao.Open()
                Dim dr As SqlDataReader = cmd.ExecuteReader()
                GridView1.DataSource = dr
                GridView1.DataBind()
            End Using
        End Using
    End Sub

    Protected Sub GridView1_RowEditing(sender As Object, e As GridViewEditEventArgs)
        GridView1.EditIndex = e.NewEditIndex
        CarregarUsuarios()
    End Sub

    Protected Sub GridView1_RowCancelingEdit(sender As Object, e As GridViewCancelEditEventArgs)
        GridView1.EditIndex = -1
        CarregarUsuarios()
    End Sub

    Protected Sub GridView1_RowUpdating(sender As Object, e As GridViewUpdateEventArgs)
        Dim id As Integer = Convert.ToInt32(GridView1.DataKeys(e.RowIndex).Value)
        Dim nome As String = DirectCast(GridView1.Rows(e.RowIndex).Cells(1).Controls(0), TextBox).Text
        Dim email As String = DirectCast(GridView1.Rows(e.RowIndex).Cells(2).Controls(0), TextBox).Text
        Dim telefone As String = DirectCast(GridView1.Rows(e


