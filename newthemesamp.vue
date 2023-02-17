<template>
	<div class="root-component">
		<!-- modal start -->
		<el-dialog :title="(input.id ? 'Edit' : 'Add') + ' Party Packages'" :visible.sync="dialogFormVisible" width="60%">
			<el-form :model="input">
				<el-form-item label="Package Name" :label-width="formLabelWidth">
					<el-input v-model="input.name_theme" auto-complete="off" placeholder="please input package name"></el-input>
				</el-form-item>
				<el-form-item label="Package Price" :label-width="formLabelWidth">
					<el-input v-model="input.price_theme" auto-complete="off" type="number" placeholder="please input package price"></el-input>
				</el-form-item>
				<el-form-item label="Min No of Kids" :label-width="formLabelWidth">
					<el-input-number v-model="input.number_kids" :min="1" :max="100"></el-input-number>
				</el-form-item>
				<el-form-item label="Additional Kids Price" :label-width="formLabelWidth">
					<el-input v-model="input.additional_price" auto-complete="off" type="number" placeholder="please input the price that will be multiplied for each additional kid above the minimum no of kids"></el-input>
				</el-form-item>			
				<!-- CXTEST. COLIN EDIT HERE -->
				<section style="padding-left:50px; padding-top:20px; padding-bottom:50px;">
					<h3 style="font-size:14px;">Deposit Amount</h3>
					<p>Define how the deposit amount is calculated for <u>this</u> party package.</p>
					<label for="depositSourceGlobal" style="display:block;">
						<input type="radio" name="partyPackageDepositSource" id="depositSourceGlobal" value="global" v-model="selectedDepositSource">
						Use global (regional) deposit amount of <strong>{{ minimumDeposit }}</strong>
					</label>
					<label for="depositSourceCustom" style="display:block;">
						<input type="radio" name="partyPackageDepositSource" id="depositSourceCustom" value="custom" v-model="selectedDepositSource">
						Use custom deposit amount of <el-input-number v-model="customMinimumDeposit" :min="0" style="margin-left:5px;"></el-input-number>
					</label>
					<!-- <label for="depositAddKids" style="display:block;">
						<input type="checkbox" name="partyPackageDepositAddKids" id="depositAddKids" value="addkids" checked="checked">
						Include price from Additional Kids in deposit amount
					</label> -->
					<!-- <label for="depositAddOns" style="display:block;">
						<input type="checkbox" name="partyPackageDepositAddOns" id="depositAddOns" v-model="includeAddOnsInDeposit">
						Include price from Selected AddOns in deposit amount
					</label> -->
				</section>
				<!-- CXTEST. COLIN EDIT HERE -->				
				<el-form-item label="Image Src" :label-width="formLabelWidth">
					<img :src="input.img_src" alt="Image" accept="image/*" class="img-fix-width" v-show="input.img_src">
				</el-form-item>
				<el-form-item label="*(318 x 180 px)" :label-width="formLabelWidth">
					<b-form-file :plain="false" @change="uploadImage($event)" ref="fileinput"></b-form-file>
				</el-form-item>
				<el-form-item label="Package Description" :label-width="formLabelWidth">
					<tinymce id="description_package" v-model="input.description_package" :ref="input.description_package" @editorInit="editorInit(input.description_package)"></tinymce>
				</el-form-item>
				<el-form-item label="Package Include" :label-width="formLabelWidth">
					<tinymce id="description_theme" v-model="input.description_theme" :ref="input.description_theme" @editorInit="editorInit(input.description_theme)"></tinymce>
				</el-form-item>
				<el-form-item label="Status" :label-width="formLabelWidth">
					<select class="form-control" v-model="input.status">
						<option value="1">Display</option>
						<option value="2">Active</option>
						<option value="0">Inactive</option>
					</select>
				</el-form-item>
			</el-form>
			<span slot="footer" class="dialog-footer">
				<el-button @click="dialogFormVisible = false">Cancel</el-button>
				<el-button type="primary" @click="save">Confirm</el-button>
			</span>
		</el-dialog>
		<!-- modal end -->
		
		<div class="animated fadeIn">
			<!-- content start -->
			<div class="row text-center">
				<div class="col-sm-12">
					<el-button type="text" @click="dialogFormVisible = true; resetInput()" v-if="$store.getters.getAccess.search('partyPackage-add') != '-1'">Add Party Package</el-button>
					<b-card>
						<table class="table table-bordered table-centered">
							<thead>
								<tr>
									<th><i class="icon-list"></i></th>
									<th>Name</th>
									<th>Price</th>
									<th>No Of Kids</th>
									<th>Additional Price</th>
									<th>Package Description</th>
									<th>Package Include</th>
									<th>Status</th>
									<th><i class="icon-options"></i></th>
								</tr>
							</thead>
							<tbody>
								<tr v-for="(t, i) in theme" :key="i">
									<td>{{ i + 1 }}</td>
									<td>{{ t.name_theme }}</td>
									<td>{{ t.price_theme }}</td>
									<td>{{ t.number_kids }}</td>
									<td>{{ t.additional_price }}</td>
									<td v-html="t.description_package"></td>
									<td v-html="t.description_theme"></td>
									<td v-if="t.status == 0">Inactive</td>
									<td v-if="t.status == 2">Active</td>
									<td v-if="t.status == 1">Display</td>
									<td class="px-0">&nbsp;
										<i class="fa fa-edit" @click="getDetail(t.id); dialogFormVisible = true; resetInput()" v-if="$store.getters.getAccess.search('partyPackage-modify') != '-1'"></i>&nbsp;
										<i class="fa fa-remove" @click="confirm(t.id)" v-if="$store.getters.getAccess.search('partyPackage-delete') != '-1'"></i>&nbsp;
									</td>
								</tr>
							</tbody>
						</table>
					</b-card>
				</div><!--/.col-->
			</div><!--/.row-->
			<!-- content end -->
		</div>
	</div>
</template>

<script>
export default {
	name: 'party-package-component',
	data: function() {
		return {
			theme: {},
			input: {
				id: 0,
				name_theme: '',
				price_theme: '',
				number_kids: 1,
				additional_theme: '',
				description_theme: '',
				description_package: '',
				status: 1,
				includeAddOnsInDeposit: false, // CXTEST
			},
			dialogFormVisible: false,
			formLabelWidth: '200px',
			minimumDeposit: '', // CXTEST
			selectedDepositSource: 'global', // CXTEST
			customMinimumDeposit: 0, // CXTEST
		}
	},
	computed: {
		depositAmount() {
			if (this.selectedDepositSource === 'global') {
				return this.minimumDeposit;
			} else {
				return this.customMinimumDeposit;
			}
		}
	},
	mounted: function() {
		this.getAll();
	},
	methods: {
		editorInit: function(value) {
			this.$refs[value].editor.setContent(value)
		},
		resetInput: function() {
			this.input.id = 0
			this.input.name_theme = ''
			this.input.price_theme = ''
			this.input.number_kids = 1
			this.input.additional_price = ''
			this.input.description_theme = ''
			this.input.description_package = ''
			this.input.custom_minimum_deposit = this.minimumDeposit // CXTEST. use global as base
			
			if (this.input.img_src_upload) {
				this.$refs.fileinput.reset()
				this.input.img_src_upload = 0
			}
		},
		getAll: function() {
			this.$http.get(this.$root.getApiPrefix() + 'new-theme').then(function (res) {
				if (this.$root.getApiCodeSuccess(res)) {
					this.theme = this.$root.getApiResponse(res)
					this.minimumDeposit = res.body.regional_minimum_deposit // CXTEST
				} else {
					this.$alert(this.$root.getApiMessage(res), 'Error', { confirmButtonText: 'OK' })
				}
			},
			function (res) {
				this.$root.showConnectionError()
			})
		},
		getDetail: function(id) {
			this.$http.get(this.$root.getApiPrefix() + 'new-theme-get-by-id?id=' + id).then(function (res) {
				this.minimumDeposit = res.body.regional_minimum_deposit // CXTEST
				if (this.$root.getApiCodeSuccess(res)) {
					this.input = this.$root.getApiResponse(res)
				} else {
					this.$alert(this.$root.getApiMessage(res), 'Error', { confirmButtonText: 'OK' })
				}
			},
			function (res) {
				this.$root.showConnectionError()
			})
		},
		save: function() {
			var method = (this.input.id ? 'put' : 'post')
			var message_alert = '';
			this.input.custom_minimum_deposit = this.depositAmount; // CXTEST
			this.$http[method](this.$root.getApiPrefix() + 'new-theme', this.input).then(function (res) {
				if (this.$root.getApiCodeSuccess(res)) {
					this.getAll()
					this.dialogFormVisible = false

					// Display success notification alert
					if (method == 'put') { message_alert = 'Edit' } else { message_alert = 'Save' }
					this.$notify({
						title: 'Success',
						message: 'Success ' + message_alert + ' Party Package',
						type: 'success'
					});
				} else {
					this.$alert(this.$root.getApiMessage(res), 'Error', { confirmButtonText: 'OK' })
				}
			},
			function (res) {
				this.$root.showConnectionError()
			})
		},
		confirm: function(id) {
			this.$confirm('Are you sure want to delete this package?', 'Warning', {
				confirmButtonText: 'OK',
				cancelButtonText: 'Cancel',
				type: 'warning'
			}).then(() => {
				this.$http.delete(this.$root.getApiPrefix() + 'new-theme?id=' + id).then(function (res) {
					if (this.$root.getApiCodeSuccess(res)) {
						this.getAll()
						this.$alert(this.$root.getApiMessage(res), 'Success', { confirmButtonText: 'OK' })
					}
					else {
						this.$alert(this.$root.getApiMessage(res), 'Error', { confirmButtonText: 'OK' })
					}
				},
				function (res) {
					this.$root.showConnectionError()
				})
			}).catch(() => {

			});
		},
		uploadImage: function(event) {
			var input = event.target;

			if (input.files && input.files[0]) {
				var reader = new FileReader();
				var vm = this

				reader.onload = function(e) {
					vm.input.img_src = e.target.result
					vm.input.img_src_upload = 1
				}
				reader.readAsDataURL(input.files[0]);
			}
		}
	}
}
</script>
