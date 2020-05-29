<template>
	<div class="device col col-xs-12 col-sm-6 col-md-3 q-pa-sm">
		<q-card bordered rounded flat>
			<q-card-section horizontal style="flex-flow: row wrap;">
				<q-card-section class="flex column justify-center">
					<q-icon :name="icon" size="xl" />
				</q-card-section>
				<q-card-section
					class="flex column justify-center"
					style="flex-grow: 1; flex-shrink: 1;"
				>
					<div class="body">{{ title }}</div>
					<div class="caption" style="color: #555;">{{ type }}</div>
				</q-card-section>
				<q-card-section class="flex column justify-center">
					<q-btn
						outline
						round
						:icon="statusIcon"
						:color="color"
						@click="toggle"
					/>
				</q-card-section>
			</q-card-section>
			<iframe v-show="false" :src="hook" frameborder="0"></iframe>
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
		mdiHelp,
		mdiPower
	} from "@mdi/js";

	export default {
		name: "Device",
		props: {
			title: {
				type: String,
				required: true
			},
			type: {
				type: String,
				required: true
			},
			action: {
				type: String,
				required: true
			},
			webhooks_key: {
				type: String,
				required: true
			}
		},
		data: () => ({
			on: null,
			hook: "",
			icons: {
				plug: mdiPowerPlug,
				plugOutline: mdiPowerPlugOutline,
				bulb: mdiLightbulb,
				bulbOutline: mdiLightbulbOutline,
				unknown: mdiHelp,
				power: mdiPower
			}
		}),
		computed: {
			icon() {
				let type = this.type === "Bulb" ? "bulb" : "plug";
				let status = this.on === true ? "" : "Outline";
				return this.icons[`${type + status}`];
			},
			statusIcon() {
				return this.on === null ? this.icons.unknown : this.icons.power;
			},
			color() {
				switch (this.on) {
					case null:
						return "warning";
						break;
					case true:
						return "primary";
						break;
					case false:
						return "secondary";
						break;
				}
			}
		},
		methods: {
			toggle() {
				let ifttt = (action, key) => {
					return `https://maker.ifttt.com/trigger/${action}/with/key/${key}`;
				};

				const action = this.action;
				const key = this.webhooks_key;

				switch (this.on) {
					case null: {
						this.on = true;
						this.hook = ifttt(action, key);
						break;
					}
					case false: {
						this.on = true;
						this.hook = ifttt(action, key);
						break;
					}
					case true: {
						this.on = false;
						this.hook = ifttt(action + "_off", key);
						break;
					}
				}
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
</style>