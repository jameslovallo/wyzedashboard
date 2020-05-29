<template>
	<q-layout
		view="lHh Lpr lFf"
		style="
		-webkit-user-select: none;"
	>
		<q-header
			elevated
			style="background: linear-gradient(to bottom, #03CCC9, #0DC4E8)"
			class="drag"
		>
			<q-toolbar>
				<q-img
					src="~assets/wyze-logo-white.svg"
					width="110px"
					class="q-mr-md q-my-md"
				/>
				<q-btn
					:icon="icons.plus"
					flat
					round
					dense
					class="no-drag"
					size="lg"
					@click="add = true"
				/>
				<q-btn
					:icon="icons.edit"
					flat
					round
					dense
					class="no-drag"
					size="lg"
					@click="edit = !edit"
				/>
				<q-space />
				<q-btn
					icon="minimize"
					flat
					round
					dense
					class="no-drag"
					@click="minimize"
				/>
				<q-btn icon="close" flat round dense class="no-drag" @click="close" />
			</q-toolbar>
		</q-header>
		<q-page-container>
			<q-page padding style="max-width: 1400px; margin: 0 auto;">
				<draggable v-model="devices">
					<transition-group class="row">
						<Device
							v-for="(device, i) in devices"
							:key="device.action"
							:action="device.action"
							:type="device.type"
							:title="device.title"
							:webhooks_key="webhooks_key"
						>
							<q-btn
								class="device-close"
								icon="close"
								color="red"
								size="xs"
								round
								@click="devices.splice(i, 1)"
								v-show="edit === true"
							/>
						</Device>
					</transition-group>
				</draggable>
				<q-dialog v-model="add">
					<q-card>
						<q-card-section class="row">
							<div class="text-h6">Add a Device</div>
							<q-space />
							<q-btn icon="close" flat round dense v-close-popup />
						</q-card-section>
						<q-separator />
						<q-card-section>
							<q-input
								label="IFTTT Webhook Action"
								v-model="newDevice.action"
								color="primary"
								filled
								style="min-width: 300px"
								class="q-mb-md"
							/>
							<q-input
								label="Device Name"
								v-model="newDevice.title"
								color="primary"
								filled
								style="min-width: 300px"
								class="q-mb-md"
							/>
							<q-select
								label="Device Type"
								:options="['Bulb', 'Plug']"
								v-model="newDevice.type"
								color="primary"
								filled
								style="min-width: 300px"
								class="q-mb-md"
							/>
							<q-btn
								label="Add Device"
								icon="add"
								color="primary"
								@click="
									devices.push(newDevice);
									newDevice = {};
									add = false;
								"
							/>
						</q-card-section>
					</q-card>
				</q-dialog>
			</q-page>
		</q-page-container>
	</q-layout>
</template>

<script>
	import draggable from "vuedraggable";
	import Device from "components/Device";
	import { mdiPlusCircleOutline, mdiPencilCircleOutline } from "@mdi/js";

	export default {
		name: "MainLayout",

		components: {
			draggable,
			Device
		},

		data() {
			return {
				add: false,
				edit: false,
				icons: {
					plus: mdiPlusCircleOutline,
					edit: mdiPencilCircleOutline
				},
				webhooks_key: "dQpcbai4uZ28My2lDWzo_N",
				newDevice: {},
				devices: [
					{
						action: "bedroom_light",
						title: "Bedroom",
						type: "Bulb"
					},
					{
						action: "bedroom_ac",
						title: "Bedroom AC",
						type: "Plug"
					},
					{
						action: "nursery_ac",
						title: "Nursery AC",
						type: "Plug"
					},
					{
						action: "living_room_light",
						title: "Living Room",
						type: "Bulb"
					}
				]
			};
		},
		methods: {
			minimize() {
				if (process.env.MODE === "electron") {
					this.$q.electron.remote.BrowserWindow.getFocusedWindow().minimize();
				}
			},
			maximize() {
				if (process.env.MODE === "electron") {
					const win = this.$q.electron.remote.BrowserWindow.getFocusedWindow();

					if (win.isMaximized()) {
						win.unmaximize();
					} else {
						win.maximize();
					}
				}
			},
			close() {
				if (process.env.MODE === "electron") {
					this.$q.electron.remote.BrowserWindow.getFocusedWindow().close();
				}
			}
		},
		created() {
			if (localStorage.devices) {
				this.devices = JSON.parse(localStorage.devices);
			}
		},
		watch: {
			devices(newDevices) {
				localStorage.devices = JSON.stringify(newDevices);
			}
		}
	};
</script>

<style>
	.drag {
		-webkit-user-select: none;
		-webkit-app-region: drag;
	}
	.no-drag {
		-webkit-user-select: none;
		-webkit-app-region: no-drag;
	}
</style>