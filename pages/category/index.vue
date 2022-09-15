<template>
  <div class="shadow-lg rounded-2xl p-4 bg-white dark:bg-gray-700 w-full">
    <el-table v-loading="loading" :data="tableData">
      <el-table-column label="Nome" prop="name" />
      <el-table-column align="right">
        <template slot="header">
          <el-button
            v-model="addNewEntity"
            size="mini"
            type="primary"
            @click="addNewEntity()"
          >Adicionar</el-button>
        </template>
        <template slot-scope="scope">

        <el-button size="mini" @click="getEntity(scope.row)">Editar</el-button>

        <el-popconfirm
          confirm-button-text='OK'
          cancel-button-text='No, Thanks'
          icon="el-icon-info"
          icon-color="red"
          title="Are you sure to delete this?"
          @confirm="handleDelete(scope.$index, scope.row)"
        >
            <el-button slot="reference" size="mini" type="danger">
              Deletar
            </el-button>
        </el-popconfirm>

        </template>
      </el-table-column>
    </el-table>

    <el-dialog :title="form.id ? 'Editar Categoria': 'Criar Categoria'" :visible.sync="dialogFormVisible">

      <el-form ref="form" :model="form" :rules="rules" >
        <el-form-item label="Nome" :label-width="formLabelWidth" prop="name">
          <el-input v-model="form.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">Cancel</el-button>
        <el-button type="primary" @click="handleConfirm()">Confirm</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>

export default {
  data() {
    return {
      tableData: [],
      loading: false,
      dialogFormVisible: false,
      formLabelWidth: '120px',
      form: {
        name: ''
      },
      rules: {
          name: [
            { required: true, message: 'Por favor, informe o nome da categoria.', trigger: 'blur' },
            { min: 3, max: 15, message: 'Tamanho deve ser 3 entre 5', trigger: 'blur' }
          ],
      },
    }
  },

  mounted() {
    this.fetchData();
  },

  methods: {
    handleConfirm(){
      if(this.form.id){
        this.handleEdit()
      }else{
        this.handleCreate()
      }
    },
    addNewEntity() {
      this.resetForm();
      this.dialogFormVisible = true
    },
    resetForm(){
        this.form = {
          name: ''
        }
    },
    getEntity(row){
      this.form = { ...row }
      this.dialogFormVisible = true
    },
    async fetchData() {
      this.loading = true;
      try {
        const { data } = await this.$axios.get("/category/get");

        this.tableData = data;
      } catch (e) {
        this.$notify.error({
          title: 'Erro',
          message: 'Não foi possível carregar'
        });
      } finally {
        this.loading = false;
      }
    },
    handleCreate(){
      this.$refs.form.validate(async (valid) => {
        if(valid){
          this.loading = true;
          try {
            await this.$axios.post(`/category/create`, this.form);

            this.$notify.success({
              title: 'Sucesso',
              message: 'Criado com sucesso'
            });


            this.dialogFormVisible = false
            this.fetchData();

          } catch (e) {
            this.$notify.error({
              title: 'Erro',
              message: 'Não foi possível criar'
            });
          }finally{
            this.loading = false;
          }
        }else{
          this.$notify.error({
              title: 'Dados inválidos',
              message: 'Não foi possível criar'
            });
        }
      })
    },
    handleEdit() {
      this.$refs.form.validate(async (valid) => {
        if(valid){
          this.loading = true;
          try {
            await this.$axios.patch(`/category/update/${this.form.id}`, this.form);

            this.$notify.success({
              title: 'Sucesso',
              message: 'Editado com sucesso'
            });

            this.resetForm();

            this.dialogFormVisible = false

            this.fetchData();

          } catch (e) {
            this.$notify.error({
              title: 'Erro',
              message: 'Não foi possível editar'
            });
          }finally{
            this.loading = false;
          }
        }else{
          this.$notify.error({
              title: 'Dados inválidos',
              message: 'Não foi possível editar'
            });
        }
      })
    },
    async handleDelete(index, row) {
      this.loading = true;
      try {
        await this.$axios.delete(`/category/delete/${row.id}`);
        this.tableData.splice(index, 1);

        this.$notify.success({
          title: 'Sucesso',
          message: 'Deletado com sucesso'
        });

      } catch (e) {
        this.$notify.error({
          title: 'Erro',
          message: 'Não foi possível deletar'
        });
      } finally {
        this.loading = false;
      }
    }
  },
}
</script>
