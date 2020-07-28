<template>
    <div class="container">
        <div class="row pt-5" v-if="$gate.isAdmin()">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                    <button type="submit" class="btn btn-success" @click="addUser">Add New <i class="fas fa-user-plus"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover text-nowrap">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>Type</th>
                      <th>Registered at</th>
                      <th>Action</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="user in users.data" :key="user.id">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td>{{user.type | upperText}}</td>
                      <td>{{user.created_at | datetime}}</td>
                      <td>
                          <a href="#" @click="editUser(user)">
                              <i class="fa fa-edit text-green"></i>
                          </a>
                          /
                          <a href="#" @click="deleteUser(user.id)">
                              <i class="fa fa-trash text-red"></i>
                          </a>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <!-- /.card-body -->
              <div class="card-footer">
                <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div>

        <div v-if="!$gate.isAdmin()">
          <not-found></not-found>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 v-if="addmode" class="modal-title" id="addNewLabel">Add New Users</h5>
                    <h5 v-else class="modal-title" id="addNewLabel">Edit Users</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <form @submit.prevent="addmode ? createUser() : updateUser()">
                <div class="modal-body">
                    <div class="form-group">
                      <label>Name</label>
                      <input v-model="form.name" type="text" name="name" placeholder="Enter your name ..."
                        class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                      <has-error :form="form" field="name"></has-error>
                    </div>
                    <div class="form-group">
                      <label>Email</label>
                      <input v-model="form.email" type="email" name="email" placeholder="Enter your email ..."
                        class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                      <has-error :form="form" field="email"></has-error>
                    </div>
                    <div class="form-group">
                      <label>Password</label>
                      <input v-model="form.password" type="password" name="password" placeholder="Enter your password ..."
                        class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                      <has-error :form="form" field="password"></has-error>
                    </div>
                    <div class="form-group">
                      <label>Type</label>
                      <select v-model="form.type" type="text" name="type" placeholder="Choose your type ..."
                        class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                        <option value=""></option>
                        <option value="admin">Admin</option>
                        <option value="user">User</option>
                        <option value="author">Author</option>
                      </select>
                      <has-error :form="form" field="type"></has-error>
                    </div>
                    <div class="form-group">
                      <label>Bio</label>
                      <textarea v-model="form.bio" type="text" name="bio" placeholder="Enter your bio ..."
                        class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                      <has-error :form="form" field="bio"></has-error>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                    <button v-if="addmode" type="submit" class="btn btn-primary">Create</button>
                    <button v-else type="submit" class="btn btn-success">Update</button>
                </div>
                </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
          return {
            addmode: true,
            users: {},
            form : new Form({
              id: '',
              name: '',
              email: '',
              password: '',
              type: '',
              bio: '',
              photo: ''
            })
          }
        },
        methods: {
          getResults(page = 1) {
            if(this.$parent.search === ""){
              axios.get('api/user?page=' + page)
              .then(response => {
              this.users = response.data;
              });
            }else{
              Fire.$emit('searching', page);
            }
          },
          addUser(){
            this.addmode = true;
            this.form.reset();
            $('#addNew').modal('show');
          },
          editUser(data){
            this.addmode = false;
            this.form.reset();
            $('#addNew').modal('show');
            this.form.fill(data);
          },
          updateUser(id){
            this.$Progress.start();
            this.form.put('api/user/'+this.form.id).then(()=>{
                toast.fire(
                  'Updated!',
                  'Your data has been updated.',
                  'success'
                );
                $('#addNew').modal('hide');
                Fire.$emit('Done');
                this.$Progress.finish();
              })
              .catch(()=>{
                this.$Progress.fail();
              });
          },
          deleteUser(id){
            toast.fire({
              title: 'Are you sure?',
              text: "You won't be able to revert this!",
              icon: 'warning',
              showCancelButton: true,
              confirmButtonColor: '#3085d6',
              cancelButtonColor: '#d33',
              confirmButtonText: 'Yes, delete it!'
            }).then((result) => {
              if(result.value){
                this.form.delete('api/user/'+id).then(()=>{
                  toast.fire(
                    'Deleted!',
                    'Your data has been deleted.',
                    'success'
                  );
                  Fire.$emit('Done');
                }).catch(()=>{
                    toast.fire(
                      'Failed!',
                      'There was something wrong.',
                      'warning'
                    );
                });
              }
            })
          },
          loadData(){
            if(this.$gate.isAdmin()){
              axios.get("api/user").then(({ data }) =>(this.users = data));
            }
          },
          createUser(){
            this.$Progress.start()
            this.form.post('api/user')
              .then(()=>{
                Fire.$emit('Done');
                $('#addNew').modal('hide');
                toast.fire({
                  icon: 'success',
                  title: 'User has been created'
                })
                this.$Progress.finish();
              })
              .catch((e)=>{
                console.log(e); 
                this.$Progress.fail();
              })
          }
        },
        created() {
            Fire.$on('searching', (page = 1) => {
              let query = this.$parent.search;
              if(query != ''){
                axios.get('api/findUser?q=' + query + '&page=' +page)
                .then((data) => {
                  this.users =  data.data;
                })
                .catch(() => {
                  this.$Progress.fail();
                });
              }else{
                this.loadData();
              }
            });
            this.loadData();
            Fire.$on('Done', () => {
              this.loadData();
            });
            // setInterval(() => this.loadData(),10000);
        }
    }
</script>
