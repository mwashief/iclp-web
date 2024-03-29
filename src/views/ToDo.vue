<template>
	<b-container>
		<h2 class="text-center mt-4">
			<b-icon icon="grip-horizontal" class="text-primary"></b-icon>
				Your Notifications
			<b-icon icon="grip-horizontal" class="text-primary"></b-icon>
		</h2>
		<hr>
		<div class="selection-box">
			<b-form-group>
				<template #label>
					<b>Select status:</b><br>
					<b-form-checkbox
					v-model="allSelected"
					:indeterminate="indeterminate"
					aria-describedby="status"
					aria-controls="status"
					@change="toggleAll"
					>
						{{ allSelected ? 'Un-select All' : 'Select All' }}
					</b-form-checkbox>
				</template>

				<template v-slot="{ ariaDescribedby }">
					<b-form-checkbox-group
						id="status"
						v-model="selected"
						:options="status"
						:aria-describedby="ariaDescribedby"
						name="status"
						class="ml-4"
						aria-label="Individual statuses"
						switches
					>
					</b-form-checkbox-group>
					<!-- <div>
						Selected: <strong>{{ selected }}</strong><br>
						All Selected: <strong>{{ allSelected }}</strong><br>
						Indeterminate: <strong>{{ indeterminate }}</strong>
					</div> -->
				</template>
			</b-form-group>
		</div>
		<hr>
		<div class="mt-5 message-box">
			<b-row>
				<b-col cols="9">
					<h4 class="text-center my-3 inv hover-zoom" @click.prevent="refresh">
						Messages 
						<b-icon 
							icon="arrow-repeat" 
							animation="throb" 
							font-scale="1" 
						>
						</b-icon>
					</h4>
				</b-col>
				<b-col cols="2">	
					<b-form-select v-model="days" :options="options" class="m-2 pl-3" @change="refresh">
						<!-- This slot appears above the options from 'options' prop -->
						<template #first>
							<b-form-select-option :value="null" disabled>-- time-span --</b-form-select-option>
						</template>
					</b-form-select>
				</b-col>
			</b-row>
			<Notifications :userId="userId" :userName="userName" :messages="messages" :selected="selected"  @choose-a-message="updateChoice"/>
		</div>
		<hr>
		<b-row
			align-h="center"
			class="mt-4 text-danger"
		>
			<b-button
				v-if="(this.choosedMessage && this.choosedMessage.status === 'full_completed')"
				pill
				variant="primary"	
				class="shadow-lg font-weight-bold"
				:disabled="choosedMessage === null"
				:to="DualResult"
			>
				See Results
			</b-button>
			<b-button
				v-else
				pill
				variant="primary"	
				class="shadow-lg font-weight-bold"
				:disabled="choosedMessage === null"
				:to="DualConfirm"
			>
				Continue to exam
			</b-button>

		</b-row>
	</b-container>
</template>

<script>
import thisuser from "@/mixins/thisuser";
import apiUtil from '@/mixins/apiUtil';
import Notifications from "@/components/Notifications";
export default {
	components: {
		Notifications
	},
	data() {
		return {
			userId: null,
			userName: null,
			messages: [	
				// {
				// 	challengee_id:3
				// 	challengee_name:"ahsanul ameen"
				// 	challenger_id:1
				// 	challenger_name:"voldemort"
				// 	exam_id:2
				// 	status:"pending"
				// 	topic_id:2
				// 	topic_name:"python",
				//	last_accessed: "timestamp"
				// },
			],
			status: ["pending", "accepted", "rejected", "half_completed", "full_completed", "archived"],
			selected: ["pending", "accepted", "rejected", "half_completed", "full_completed", "archived"],
			allSelected: true,
			indeterminate: false,
			choosedMessage: null,
			days: 1,
			options: [
				{ value: 1, text: 'last day' },
				{ value: 7, text: 'last week' },
				{ value: 30, text: 'last month' },
				{ value: 365, text: 'last year' }
			]
		};
	},
	methods: {
		refresh() {
			console.log("refreshing messages...");
			
			this.apiPostPromise("/dual/archive")
			.then(() => {
				console.log("outdated pending message archived...");
			})
			.catch (() => {
				this.$root.$bvToast.toast("ERROR! dual/archive failed", {
					variant: "danger",
					autoHideDelay: 2000,
					appendToast: true,
					noCloseButton: true,
					toaster: "b-toaster-top-center",
				});
			});

			this.apiGet(`/dual/invitations?userid=${this.userId}&days=${this.days}`)
			.then((data) => {
				this.messages = data;
				//this.printJsonArray(this.messages);
			});
		},
		setUser(id, name) {
			this.userId = id;
			this.userName = name;
		},
		printJsonArray(jarray) {
			console.log("-----printing json array---------");
			for (var i = 0; i < jarray.length; i++) {
				console.log( jarray[i] );
			}
			console.log("---------------------------------");
		},
		// pass this to the event-handler of the component
		updateChoice(message) {
			this.choosedMessage = message;
		},
		toggleAll(checked) {
			this.selected = checked ? this.status.slice() : []
		},
	},
	computed: {
		DualConfirm() {
			return {
				name: 'DualConfirm',
				params: { 
					topicId: this.choosedMessage ? this.choosedMessage.topic_id : null,
					topicName: this.choosedMessage ? this.choosedMessage.topic_name : null,
					challengerName: this.choosedMessage? this.choosedMessage.challenger_name: null,
					challengerId: this.choosedMessage? this.choosedMessage.challenger_id: null,
					challengeeId: this.choosedMessage? this.choosedMessage.challengee_id: null,
					challengeeName: this.choosedMessage? this.choosedMessage.challengee_name: null,
					throwingType: false,
					examId: this.choosedMessage? this.choosedMessage.exam_id: null,
					status: this.choosedMessage? this.choosedMessage.status: null
				} 
			}
		},
		DualResult() {
			return {
				name: 'DualResult',
				params: { 
					idOne: this.choosedMessage? this.choosedMessage.challenger_id: null,
					idTwo: this.choosedMessage? this.choosedMessage.challengee_id: null,
					examId: this.choosedMessage? this.choosedMessage.exam_id: null,
				}
			}
		},
	},
	mounted() {
		
		this.refresh();
	},
	created() {
		this.setUser(this.userid, this.username);
	},
	watch: {
		selected(newValue, oldValue) {
			console.log(oldValue);
			// Handle changes in individual status checkboxes
			if (newValue.length === 0) {
				this.indeterminate = false
				this.allSelected = false
			} else if (newValue.length === this.status.length) {
				this.indeterminate = false
				this.allSelected = true
			} else {
				this.indeterminate = true
				this.allSelected = false
			}
		}
	},
	mixins: [thisuser, apiUtil],
};
</script>

<style lang="scss" scoped>
	.selection-box {
		border-radius: 10px;
		border: 1px solid #ffab73;
		background-color: rgb(251, 247, 242);
		padding: 10px;
	}
	.message-box {
		border-radius: 25px;
		border: 2px solid #0e0b09;
		background-color: rgb(67, 40, 70);
	}
	.inv {
		color: rgb(218, 255, 148);
	}
</style>

