<template>
	<div class="device col col-xs-12 col-sm-6 col-md-3 q-pa-sm">
		<q-card bordered rounded flat>
			<div v-if="feed" class="feed">
				<webview
					class="video"
					:src="feed"
					webpreferences="allowRunningInsecureContent"
					disablewebsecurity
					:partition="Math.random()"
				></webview>
			</div>
			<q-item class="q-px-sm">
				<q-item-section avatar>
					<q-icon :name="icon" color="grey-10" size="34px" />
				</q-item-section>
				<q-item-section>
					<q-item-label lines="1">
						<span class="text-grey-10">{{ title }}</span>
					</q-item-label>
					<q-item-label caption lines="1">
						{{ type }}
					</q-item-label>
				</q-item-section>
				<q-item-section side>
					<div class="text-primary q-gutter-xs">
						<q-btn
							v-if="on"
							size="12px"
							flat
							dense
							label="on"
							@click="webhook(on)"
						/>
						<q-btn
							v-if="off"
							size="12px"
							flat
							dense
							label="off"
							@click="webhook(off)"
						/>
						<q-btn
							v-if="record"
							size="md"
							flat
							round
							color="red"
							:icon="icons.record"
							@click="
								webhook(record);
								showNotif();
							"
						/>
					</div>
				</q-item-section>
			</q-item>
		</q-card>
		<slot />
	</div>
</template>

<script>
	import {
		mdiPowerPlugOutline,
		mdiPowerPlug,
		mdiLightbulb,
		mdiLightbulbOutline,
		mdiVideo,
		mdiVideoOutline,
		mdiHelp,
		mdiPower,
		mdiRecordRec
	} from "@mdi/js";

	export default {
		name: "Device",
		props: ["title", "type", "on", "off", "record", "feed", "webhooks_key"],
		data: () => ({
			icons: {
				plug: mdiPowerPlug,
				plugOutline: mdiPowerPlugOutline,
				bulb: mdiLightbulb,
				bulbOutline: mdiLightbulbOutline,
				camera: mdiVideo,
				cameraOutline: mdiVideoOutline,
				unknown: mdiHelp,
				power: mdiPower,
				record: mdiRecordRec
			}
		}),
		computed: {
			icon() {
				const iconIndex = {
					Bulb: "bulb",
					Plug: "plug",
					Camera: "camera"
				};
				let type = iconIndex[`${this.type}`];
				let status = this.on === true ? "" : "Outline";
				return this.icons[`${type + status}`];
			}
		},
		methods: {
			webhook(action) {
				const key = this.webhooks_key;
				fetch(`https://maker.ifttt.com/trigger/${action}/with/key/${key}`, {
					method: "POST"
				});
			},
			showNotif() {
				this.$q.notify({
					message: `Recording triggered for camera "${this.title}"`,
					icon: this.icons.record,
					classes: "video-toast bg-grey-10"
				});
			}
		}
	};
</script>

<style>
	.device {
		position: relative;
	}

	.device .device-close {
		position: absolute;
		top: -0px;
		right: -0px;
	}

	.feed {
		position: relative;
		padding-top: calc(9 / 16 * 100%);
	}

	.video {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	.video-toast svg {
		fill: red;
		height: 36px;
		width: 36px;
	}
</style>