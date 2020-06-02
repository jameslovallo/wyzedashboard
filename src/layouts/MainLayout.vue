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
			<q-toolbar height="60px">
				<q-img
					src="~assets/wyze-logo-white.svg"
					width="110px"
					class="q-mr-md"
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
				<draggable v-model="cameras">
					<transition-group class="row">
						<Device
							v-for="(device, i) in cameras"
							:key="device.feed"
							:action="device.action"
							:feed="device.feed"
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
								@click="cameras.splice(i, 1), (edit = false)"
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
						<q-tabs v-model="deviceTab">
							<q-tab name="Bulb" label="Bulb" />
							<q-tab name="Plug" label="Plug" />
							<q-tab name="Camera" label="Camera" />
						</q-tabs>
						<q-tab-panels v-model="deviceTab">
							<q-tab-panel v-for="d in ['Bulb', 'Plug']" :key="d" :name="d">
								<q-input
									label="Device Name"
									v-model="newDevice.title"
									color="primary"
									filled
									style="min-width: 300px"
									class="q-mb-md"
								/>
								<q-input
									label="IFTTT Webhook Action"
									v-model="newDevice.action"
									color="primary"
									filled
									style="min-width: 300px"
									class="q-mb-md"
								/>
								<q-btn
									label="Add Device"
									icon="add"
									color="primary"
									@click="addDevice"
								/>
							</q-tab-panel>
							<q-tab-panel name="Camera">
								<q-input
									label="Device Name"
									v-model="newDevice.title"
									color="primary"
									filled
									style="min-width: 300px"
									class="q-mb-md"
								/>
								<q-input
									label="IPCamLive Camera uRL"
									v-model="newDevice.feed"
									color="primary"
									filled
									style="min-width: 300px"
									class="q-mb-md"
								/>
								<p class="caption text-grey-8">
									Your camera's "live" tab url on
									<a href="www.ipcamlive.com">ipcamlive.com</a>. To connect your
									camera to ipcamlive, you will need to install the
									<a
										href="https://support.wyzecam.com/hc/en-us/articles/360026245231-Wyze-Cam-RTSP"
										>Wyze RTSP firmware</a
									>
									on your camera and forward port 554 on your router for your
									camera's IP address.
								</p>
								<q-input
									label="IFTTT Webhook Action (Optional)"
									v-model="newDevice.action"
									color="primary"
									filled
									style="min-width: 300px"
									class="q-mb-md"
								/>
								<q-btn
									label="Add Device"
									icon="add"
									color="primary"
									@click="addDevice"
								/>
							</q-tab-panel>
						</q-tab-panels>
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
				deviceTab: "Bulb",
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
				],
				cameras: []
			};
		},
		methods: {
			addDevice() {
				this.newDevice.type = this.deviceTab;
				this.newDevice.type === "Camera"
					? this.cameras.push(this.newDevice)
					: this.devices.push(this.newDevice);
				this.newDevice = {};
				this.add = false;
			},
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
			if (localStorage.cameras) {
				this.cameras = JSON.parse(localStorage.cameras);
			}
		},
		watch: {
			devices(newDevices) {
				localStorage.devices = JSON.stringify(newDevices);
			},
			cameras(newCameras) {
				localStorage.cameras = JSON.stringify(newCameras);
			}
		},
		mounted() {
			let body = "body {overflow: hidden !important;}";
			let playerdiv =
				"#playerdiv {position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 1000;}";
			let playeriframe =
				"#playeriframe {position: absolute; top: 0; left: 0; height: 100% !important; width: 100% !important;}";
			document.querySelectorAll(".video").forEach(el => {
				el.addEventListener("dom-ready", function() {
					el.insertCSS(body + playerdiv + playeriframe);
					el.style.pointerEvents = "none";
				});
			});

			document.querySelectorAll(".feed").forEach(el => {
				el.addEventListener("click", () => {
					el.classList.toggle("full");
				});
			});
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
	.full {
		position: fixed;
		top: 50px;
		right: 0;
		bottom: 0;
		left: 0;
	}
</style>