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
					:style="mac ? 'margin-left: 70px' : ''"
				/>
				<q-space v-if="mac" />
				<q-btn
					icon="settings"
					flat
					round
					dense
					class="no-drag"
					@click="settings = true"
				/>
				<q-space v-if="!mac" />
				<q-btn
					v-if="!mac"
					icon="minimize"
					flat
					round
					dense
					class="no-drag"
					@click="minimize"
				/>
				<q-btn
					v-if="!mac"
					icon="close"
					flat
					round
					dense
					class="no-drag"
					@click="close"
				/>
			</q-toolbar>
		</q-header>
		<q-page-container>
			<q-page padding style="max-width: 1400px; margin: 0 auto;">
				<draggable v-model="devices">
					<transition-group class="row">
						<Device
							v-for="(device, i) in devices"
							:key="device.title"
							:on="device.on"
							:off="device.off"
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
								@click="
									devices.splice(i, 1);
									edit = false;
								"
								v-show="edit === true"
							/>
						</Device>
					</transition-group>
				</draggable>
				<draggable v-model="cameras">
					<transition-group class="row">
						<Device
							v-for="(device, i) in cameras"
							:key="device.title"
							:record="device.record"
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
								@click="
									cameras.splice(i, 1);
									edit = false;
								"
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
						<q-tabs v-model="deviceTab" indicator-color="primary">
							<q-tab name="Bulb" label="Bulb" />
							<q-tab name="Plug" label="Plug" />
							<q-tab name="Camera" label="Camera" />
						</q-tabs>
						<q-separator />
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
									label="On (IFTTT webhook action)"
									v-model="newDevice.on"
									color="primary"
									filled
									style="min-width: 300px"
									class="q-mb-md"
								/>
								<q-input
									label="Off (IFTTT webhook action)"
									v-model="newDevice.off"
									color="primary"
									filled
									style="min-width: 300px"
									class="q-mb-md"
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
									label="IPCamLive Camera URL"
									v-model="newDevice.feed"
									color="primary"
									filled
									style="min-width: 300px"
									class="q-mb-md"
								/>
								<q-input
									label="Record a Clip (IFTTT webhook action)"
									v-model="newDevice.record"
									color="primary"
									filled
									style="min-width: 300px"
									class="q-mb-md"
								/>
							</q-tab-panel>
						</q-tab-panels>
						<q-card-actions>
							<q-btn
								label="Add Device"
								icon="add"
								color="primary"
								flat
								@click="addDevice"
							/>
							<q-space />
							<q-btn
								label="Help"
								icon="help"
								flat
								dense
								@click="nativeLink('')"
							/>
						</q-card-actions>
					</q-card>
				</q-dialog>
				<q-dialog v-model="settings">
					<q-card>
						<q-card-section class="row">
							<div class="text-h6">Settings</div>
							<q-space />
							<q-btn icon="close" flat round dense v-close-popup />
						</q-card-section>
						<q-card-section>
							<q-input
								label="IFTTT Webhook Key"
								v-model="webhooks_key"
								color="primary"
								filled
								style="min-width: 300px"
							/>
						</q-card-section>
						<q-card-actions>
							<q-btn
								label="Export Devices"
								flat
								color="primary"
								@click="exportDevices"
							/>
							<q-btn
								label="Import Devices"
								flat
								color="primary"
								@click="importDevices"
							/>
						</q-card-actions>
					</q-card>
				</q-dialog>
			</q-page>
			<q-fab
				v-model="fab"
				v-if="!edit"
				vertical-actions-align="right"
				color="primary"
				icon="add"
				direction="up"
				style="position: absolute; bottom: 24px; right: 24px;"
			>
				<q-fab-action
					label-position="left"
					color="primary"
					@click="add = true"
					icon="add"
					label="Add a Device"
				/>
				<q-fab-action
					label-position="left"
					color="red"
					@click="edit = !edit"
					icon="delete"
					label="Remove a Device"
				/>
			</q-fab>
			<q-fab
				v-if="edit"
				@click="edit = false"
				color="red"
				icon="close"
				label="Cancel"
				style="position: absolute; bottom: 24px; right: 24px;"
			/>
		</q-page-container>
	</q-layout>
</template>

<script>
	import draggable from "vuedraggable";
	import Device from "components/Device";

	const jsonfile = require("jsonfile");

	const fs = require("fs");
	const homedir = require("os").homedir();
	const { dialog } = require("electron").remote;
	const { shell } = require("electron");

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
				settings: false,
				fab: false,
				deviceTab: "Bulb",
				webhooks_key: "",
				newDevice: {},
				devices: [],
				cameras: []
			};
		},
		methods: {
			addDevice() {
				this.newDevice.type = this.deviceTab;
				this.newDevice.type === "Camera"
					? this.cameras.push(this.newDevice) && location.reload()
					: this.devices.push(this.newDevice);
				this.newDevice = {};
				this.add = false;
			},
			nativeLink(link) {
				shell.openExternal(link);
			},
			async exportDevices() {
				let dir = await dialog.showOpenDialog({
					properties: ["openDirectory"],
					defaultPath: homedir
				});

				if (dir != undefined) {
					let path = await dir.filePaths;
					let settings = {
						devices: this.devices,
						cameras: this.cameras,
						webhooks_key: this.webhooks_key
					};

					const file = `${path}/wyze_devices.json`;
					jsonfile.writeFile(file, settings, function(err) {
						if (err) console.error(err);
					});

					this.$q.notify({
						message: `Saved device settings to ${path}${
							process.platform === "win32" ? "\\" : "/"
						}devices.wyze`,
						classes: "bg-grey-10"
					});
				}
			},
			async importDevices() {
				let file = await dialog.showOpenDialog({
					defaultPath: homedir,
					filters: [{ name: "JSON Files", extensions: ["json"] }]
				});

				if (file != undefined) {
					let data;
					jsonfile
						.readFile(file.filePaths[0])
						.then(obj => {
							this.devices = obj.devices;
							this.cameras = obj.cameras;
							this.webhooks_key = obj.webhooks_key;
						})
						.catch(error => console.error(error));
				}
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
		computed: {
			mac() {
				return this.$q.platform.is.platform === "mac";
			}
		},
		created() {
			if (localStorage.devices) {
				this.devices = JSON.parse(localStorage.devices);
			}
			if (localStorage.cameras) {
				this.cameras = JSON.parse(localStorage.cameras);
			}
			if (localStorage.webhooks_key) {
				this.webhooks_key = localStorage.webhooks_key;
			}
		},
		watch: {
			devices(newDevices) {
				localStorage.devices = JSON.stringify(newDevices);
			},
			cameras(newCameras) {
				localStorage.cameras = JSON.stringify(newCameras);
			},
			webhooks_key(newWebhooks_key) {
				localStorage.webhooks_key = newWebhooks_key;
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

				setInterval(() => {
					el.reload();
				}, 240000);
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
		z-index: 1000;
	}
</style>