<template>
	<div>
		<header class="col-12">
			<div>{{ character.name }}</div>
		</header>
		
		<!-- <teleport v-if="character.splat !== EnumSplat.VAMPIRE" :to="`#${EnumSplat[character.splat].toLowerCase()}-conditions`"> -->
		<!-- </teleport> -->
		<div class="charsheet" :class="{
			['charsheet-'+(EnumSplat && EnumSplat[character.splat]).toLowerCase()]: true
		}">
			<!-- <fab /> -->

			<div class="fab">
				<button @click="rollSelected()">Roll Selected</button>
			</div>

			{{selectedTraits}}

			<div id="infoBar" class="bar row col-12">

				<!-- <datalist> </datalist> -->

				<div class="block col-sm-4">
					{{ splat && splat.nameName }}: <input v-model="character.name" /><br />
					<span v-if="character.splat === EnumSplat.MORTAL">
						Age: <input v-model.number="character.age" type="number" /><br />
						Player: <input v-model="character.player" /><br />
					</span>
					<span v-else>
						Player: <input v-model="character.player" /><br />
						Chronicle: <input v-model="character.chronicle" /><br />
					</span>
				</div>
				<div class="block col-sm-4">
					{{ splat && splat.virtueAnchorName }}:
					<input v-model="character.virtueAnchor" list="virtueAnchors" /><br />
					{{ splat && splat.viceAnchorName }}:
					<input v-model="character.viceAnchor" list="viceAnchors" /><br />
					Concept: <input v-model="character.concept" /><br />
				</div>
				<div class="block col-sm-4">
					<span v-if="character.splat === EnumSplat.MORTAL">
						{{ splat && splat.subTypeName }}:
						<input v-model="character.chronicle" /><br />
						{{ splat && splat.legacyName }}:
						<input v-model="character.faction" /><br />
					</span>
					<span v-else>
						{{ splat && splat.subTypeName }}:
						<input v-model="character.subType" list="subTypes" /><br />
						{{ splat && splat.legacyName }}:
						<input v-model="character.legacy" /><br />
					</span>

					{{ splat && splat.orgName }}:
					<input v-model="character.organization" list="organizations"/><br />

					<datalist id="organizations">
						<option v-for="org in splat.organizations" :key="org.name">{{ org.name }}</option>
					</datalist>

					<datalist id="subTypes">
						<option v-for="el in splat.subTypes" :key="el.name">{{ el.name }}</option>
					</datalist>

					<datalist id="virtueAnchors">
						<option v-for="el in splat.virtueAnchors" :key="el">{{ el }}</option>
					</datalist>


					<datalist id="viceAnchors">
						<option v-for="el in splat.viceAnchors" :key="el">{{ el }}</option>
					</datalist>
				</div>
			</div>
			<div class="separator col-12"><h2>Attributes</h2></div>
			<div id="attrBar" class="bar row col-12">
				<div id="attr-cats" style="text-align:right;" class="block col-sm-2">
					Power<br />
					Finesse<br />
					Resistance<br />
				</div>
				<div class="attr-proper row col-sm-10">
					<div v-for="attrCat in attributeNames" :key="attrCat" class="block col-sm-4">
						<span style="text-transform: capitalize" v-for="attr in attrCat" :key="attr">
							<span :class="{'selected': selectedTraits[attr]}" @click="selectTrait(attr,{attr:true})">{{ attr }}</span>
							<input v-if="!dotsOverFive && attrMax > 5" v-model.number="character.baseAttributes[attr]" type="number" style="width: 35px;" class="attr-input">
							<div class="sheet-dots">
								<button @click="setAttr(attr, n)" v-for="n in dotAttrMax" :key="n" :class="{'sheet-dot':true,'sheet-dot-full':character.baseAttributes[attr]>=n,'sheet-dot-small':dotAttrMax>5}"></button>
							</div>
							<br>
						</span>
					</div>
				</div>		
			</div>
			<div class="row col-12">
				<div id="skills" class="bar col col-sm-4">
					<h2 class="separator col-sm-12" style="margin-bottom: 20px">Skills</h2>
					<div v-for="(cat,i) in skills" :key="cat" class="block col col-12">
						<h3 class="separator">{{Object.keys(skillCats)[i]}}</h3>
						<i class="col-12 subtitle">({{ Object.values(skillCats)[i] }} unskilled)</i><br>

						<div style="font-style: italic;font-size: 10px;line-height: 10px;" v-if="character.splat === EnumSplat.MAGE">
							Rote<br/>Skill
						</div>
						<span style="text-transform: capitalize" v-for="skill in cat" :key="skill">
							<!-- <button v-if="character.splat === EnumSplat.MAGE" @click="toggleRoteSkill(skill)" class="sheet-box" :class="{'sheet-dot-full': character.roteSkills.includes(skill)}"></button> -->
							<button v-if="character.splat === EnumSplat.MAGE" class="sheet-box" :class="{'sheet-dot-full': organization.skills.includes(skill)}"></button>
							<span :class="{'selected': selectedTraits[skill],'specialties': character.specialties[skill] && character.specialties[skill].length > 0}" @click="selectTrait(skill,{skill:true,skillCat: i})">
								{{ skill.replaceAll("_", " ") }}
							</span>
							
							<button class="dropdown-toggle material-icons" @click="specialtyDropDown(skill)">
								<span v-if="specialtyDropSelect === skill">arrow_drop_down</span>
								<span v-else>arrow_right</span>
							</button>

							<div class="sheet-dots">
								<button @click="setSkill(skill, n)" v-for="n in dotAttrMax" :key="n" class="sheet-dot" :class="{'sheet-dot-full': character.skills[skill] >= n,'sheet-dot-small':dotAttrMax>5}"></button>
							</div>
							<br>

							<item-list v-if="specialtyDropSelect === skill" class="col-12" :items="character.specialties[skill]" :mutable="true"  />

						</span>
					</div>
					<br>		
				</div>

				<div id="traits" class="bar col-sm-8">
					<h2 class="separator col-sm-12" style="margin-bottom: 20px">Other Traits</h2>
					<div class="row col-12">
						<div class="col col-sm-7">

							<ability-list v-if="splat && splat.abilityName" :abilities="character.abilities" :optionsMutable="!splat.finiteAbilities" :abilityName="splat.abilityName" id="ability" class="block" />

							<ability-list :abilities="character.merits" abilityName="Merits" id="merits" class="block" />

							<!-- <div id="werewolf-conditions"> </div> -->
							<item-list v-if="character.splat === EnumSplat.WEREWOLF" class="col-12" name="Conditions" :items="character.conditions" :mutable="true"  />


							<div id="minorTraits" class="block col col-12">
								<span v-if="character.splat !== EnumSplat.WEREWOLF">
									Size: <input v-model.number="character.size" type="number" /><br>
									Speed: {{ character.speed }}<br>
									Defense: {{ character.defense }}<br>
									<!-- Armor: <input v-model="character.armor" /><br> -->
									Armor: {{ character.armor ? character.armor.general+"/"+character.armor.ballistic : "" }}<br>
									Initative Mod: {{ initative }}<br>
								</span>
								<span style="float:left; margin-right: 5px;">Beats:</span>
								<div style="float:left; margin-right: 10px;">
									<span v-for="n in 5" :key="n">
										<button class="sheet-box" @click="setTrait('beats', n)" :class="{'sheet-dot-full': character.beats >= n}"></button>
									</span>
								</div>
								<span style="clear:both;"></span>
								<br v-if="character.splat !== EnumSplat.WEREWOLF">
								Experience: <input v-model.number="character.experience" type="number" /><br>
								<div v-if="(splat && splat.alternateBeatName) && !splat.alternateBeatOptional">
									<span style="float:left;">{{ splat.alternateBeatName }} Beats:</span> 
									<div style="float:left;">
										<span v-for="n in 5" :key="n">
											<button class="sheet-box" @click="setTrait('alternateBeats', n)" :class="{'sheet-dot-full': character.alternateBeats >= n}"></button>
										</span>
										</div>
										<span style="clear:both;"></span>
										<br>
									{{ splat.alternateBeatName }} Experience: <input v-model.number="character.alternateExperience" type="number" /><br>
								</div>
							</div>

						</div>
						<div class="col col-sm-5">
							<health-component id="health" class="col-12" style="margin-bottom: 15px" :maxHealth="character.maxHealth" :healthTrack="character.healthTrack" name="Health">
								<!-- <i class="subtitle" style="margin-bottom: 5px" v-if="character.splat === EnumSplat.WEREWOLF">(EEE)</i> -->
							</health-component>

							<div id="willpower" class="col-12" style="margin-bottom: 15px">
								<h3 class="separator col-sm-12">Willpower</h3>
								<div class="sheet-dots" style="margin-top:-10px;">
									<button v-for="n in maxWillpower" :key="n" @click="setTrait('spentWillpowerDots', maxWillpower-n, {off:-1})" class="sheet-dot" :class="{'sheet-dot-full': maxWillpower-character.spentWillpowerDots >= n}"></button>
								</div>
								<div class="sheet-boxes" :style="{'margin-left': (character.spentWillpowerDots*-15)+'px'}" style="margin-top:-7px;">
									<button v-for="n in maxWillpower-character.spentWillpowerDots" :key="n" class="sheet-box" @click="setTrait('willpower', n)" :class="{'sheet-dot-full': character.willpower >= n}"></button>
								</div>							
							</div>
							<div v-if="splat && splat.powerTraitName" class="col-12" id="powerTrait" style="margin-bottom: 15px">
								<h3 :class="{'selected': selectedTraits['power']}" @click="selectTrait('power', {})" class="separator col-sm-12">{{ splat.powerTraitName }}</h3>
								<div class="sheet-dots" style="margin-top:-10px;">
									<button v-for="n in 10" :key="n" class="sheet-dot" @click="setTrait('power', n, {min: 1})" :class="{'sheet-dot-full': character.power >= n}"></button>
								</div>
							</div>						
							<div v-if="splat && splat.fuelTraitName" class="col-12" id="fuelTrait" style="margin-bottom: 15px">
								<h3 class="separator col-sm-12">{{ splat.fuelTraitName }}</h3>
								<div class="sheet-boxes" style="margin-top:-10px;">
									<span v-for="n in maxFuel" :key="n">
										<button class="sheet-box" @click="setTrait('fuel', n)" :class="{'sheet-dot-full': character.fuel >= n}"></button>
										<br v-if="n%10===0">
									</span>
								</div>
							</div>
							<integrity-component v-if="splat && splat.integrityTraitName" :character="character" :splat="splat" class="col-12" id="integrityTrait" />
						
							<div class="col-12" v-if="character.splat===EnumSplat.WEREWOLF" style="color: black;" id="kuruth-triggers">
								<h3 class="separator col-sm-12">Kuruth Trigers</h3>
								<!-- <div class="row col-12"> -->
									<!-- <div style="color:black;" class="col-12"> -->
								Passive: <br>
								<input class="line" style="width:70%;" v-model="character.kuruthTriggers.passive"><br>
								Common: <br>
								<input class="line" style="width:70%;" v-model="character.kuruthTriggers.common"><br>
								Specific: <br>
								<input class="line" style="width:70%;" v-model="character.kuruthTriggers.specific">
									<!-- </div> -->
								<!-- </div> -->
							</div>
							
							<item-list v-if="character.splat !== EnumSplat.WEREWOLF" class="col-12" name="Conditions" :items="character.conditions" :mutable="true"  />
						</div>
					</div>
				</div>

				<div v-if="character.splat === EnumSplat.MAGE" id="mage-traits" class="row col-12">
					<div class="col-sm-4">
							<div class="col-12" id="activeSpells" style="margin-bottom: 15px">
								<h4 class="separator col-sm-12">Active Spells</h4>
								<input class="line" v-for="n in character.power" :key="n" v-model="character.activeSpells[n-1]">
							</div>
							<item-list id="yantras" class="col-12" name="yantras" :items="character.yantras" :mutable="true" style="margin-bottom: 15px"  />
							<item-list id="tools" class="col-12" name="magical tools" :items="character.magicalTools" :mutable="true"  style="margin-bottom: 15px"/>
							<item-list id="praxes" class="col-12" name="praxes" :items="character.praxes" :mutable="true"  style="margin-bottom: 15px"/>
							<item-list id="inuredSpells" class="col-12" name="inured spells" :items="character.inuredSpells" :mutable="true"  style="margin-bottom: 15px"/>
					</div>

					<div class="col-sm-8">
							<object-list :items="rotes" name="Rotes"></object-list>
							<!-- <div id="rotes" class="row col-sm-12">
								<h4 class="separator col-sm-12" style="margin-bottom: 15px">Rotes</h4>
									<div class="col-2" >
										<i class="subtitle">Arcanum</i>
									</div>
									<div id="rote-level" class="col-1">
										<i class="subtitle">Level</i>
									</div>
									<div class="col-5">
										<i class="subtitle">Spell</i>
									</div>
									<div class="col-2">
										<i class="subtitle">Creator</i>
									</div>
									<div class="col-2">
										<i class="subtitle">Rote Skill</i>
									</div>
								<div @input="doInputRote(rote,i)" class="row" v-for="(rote,i) in rotes" :key="i">
									<div class="col-2" >
										<input class="line"  v-model="rote.arcanum">
									</div>
									<div id="rote-level" class="col-1">
										<input class="line" v-model.number="rote.level" type="number">
									</div>
									<div class="col-5">
										<input class="line" v-model="rote.spell">
									</div>
									<div class="col-2">
										<input class="line" v-model="rote.creator">
									</div>
									<div class="col-2">
										<input style="text-transform: capitalize;" class="line" v-model="rote.roteSkill">
									</div>
								</div>
							</div> -->
							<br>
							<object-list :items="character.weapons"></object-list>

					</div>
				</div>

				<div id="werewolf-forms" class="row col w-100" v-if="character.splat===EnumSplat.WEREWOLF">
					<div v-for="(form) in splat.forms"
						:key="form" style="text-align:left;width:100%" class="col-sm">
						<h4 @click="character.currentForm = form.name.toLowerCase()" :class="{'form-active': character.currentForm.toLowerCase() === form.name.toLowerCase()}" class="separator col-sm-12">{{ form.name }}</h4>
						<i class="subtitle">({{ form.desc }})</i>
						
						<div>
							Strength: {{ character.attributes.strength         - currentForm.strengthMod     + form.strengthMod }}<br>
							Dexterity: {{ character.attributes.dexterity       - currentForm.dexterityMod    + form.dexterityMod }}<br>
							Stamina: {{ character.attributes.stamina           - currentForm.staminaMod      + form.staminaMod }}<br>
							Manipulation: {{ character.attributes.manipulation - currentForm.manipulationMod + form.manipulationMod }}<br>

							<br>

							Size: <input v-if="form.name==='Hishu'" v-model.number="sizeMinusForm" type="number" />
							<span v-else>{{ character.size - currentForm.sizeMod + form.sizeMod }}</span><br>
							Defense: {{ formDefense(form) }}<br>
							Initative: {{ initative - currentForm.dexterityMod + form.dexterityMod }}<br>
							<!-- Armor: <input v-model="character.armor" /><br> -->
							Speed: {{ character.speed - currentForm.strengthMod - currentForm.dexterityMod - currentForm.speedMod + form.strengthMod + form.dexterityMod + form.speedMod }}<br>
							Armor: {{ character.armor ? character.armor.general+"/"+character.armor.ballistic : "" }}<br>
							Perception: {{ perception - currentForm.perceptionMod + form.perceptionMod }} <br>
							<span v-if="form.name==='Gauru'">Kuruth Limit: {{ character.attributes.stamina - currentForm.staminaMod + character.power }}</span><br>

							<div style="line-height: 15px;" class="form-traits">
								<span v-for="(trait,i) in form.traits" :key="i">
									<br >
									<i class="subtitle" >{{trait}}</i>
								</span>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>


	</div>
</template>

<script lang="ts">
import { computed, defineComponent, ref, Ref, toRefs } from "vue";

import { Splat, SPLATS, EnumSplat, Form } from "../definitions/Splat";
import Character, { Ability, Attributes } from "../definitions/Character";

import AbilityList from "@/components/sheetComponents/AbilityList.vue";
import HealthComponent from "@/components/sheetComponents/HealthComponent.vue";
import IntegrityComponent from "@/components/sheetComponents/IntegrityComponent.vue";
import ItemList from "@/components/sheetComponents/ItemList.vue";
import ObjectList from "@/components/sheetComponents/ObjectList.vue";

// import fab from "vue-fab";

import deepmerge from "deepmerge";

import RandomUtil from "../RandomUtil";

// <div class="sheet-dots">
// 	<button @click="setAttr('intelligence', n)" v-for="n in attrMax" :key="n" class="sheet-dot"></button>
// </div>

interface CharacterViewData {
	characters: { [index: string]: Character };
	character: Character;
	attributeNames: string[][];
	skills: string[][];
	skillCats: {[index: string]: number};
	attributes: Attributes;
}

const x = defineComponent({
	name: "CharacterView",
	components: {
		"AbilityList": AbilityList,
		"HealthComponent": HealthComponent,
		"IntegrityComponent": IntegrityComponent,
		"ItemList": ItemList,
		"ObjectList": ObjectList
		// "fab": fab
	},
	computed: {
		id (): string {
			return this.$route.params.id as string;
		},
		splat (): Splat {
			// console.log(EnumSplat[this.character.splat]);
			return SPLATS[this.character.splat];
		},
		subType() {
			return (this as any).splat.subTypes.filter((el: any)=>el.name.toLowerCase() === this.character.subType.toLowerCase())[0] || {};
		},
		organization() {
			return (this as any).splat.organizations.filter((el: any)=>el.name.toLowerCase() === this.character.organization.toLowerCase())[0] || {skills:[]};
		},
		// attributes(): Attributes {
		// 	return {
		// 		intelligence: this.character.attributes.intelligence,
		// 		wits: this.character.attributes.wits,
		// 		resolve: this.character.attributes.resolve,

		// 		strength : this.character.attributes.strength  + this.getNum("this.currentForm.strengthMod") + this.getNum("this.character.abilities.vigor.level"),
		// 		dexterity: this.character.attributes.dexterity + this.getNum("this.currentForm.dexterityMod"),
		// 		stamina  : this.character.attributes.stamina   + this.getNum("this.currentForm.staminaMod")  + this.getNum("this.character.abilities.resilience.level"),

		// 		presence: this.character.attributes.presence,
		// 		manipulation: this.character.attributes.manipulation + this.getNum("this.currentForm.manipulationMod"),
		// 		composure: this.character.attributes.composure,
		// 	};
		// },		
		attrMax() {
			// console.log(this.splat);
			return (this as any).character.power > 5 ? ((this as any).character as Character).power : 5;
		},
		dotAttrMax() {
			return Math.min((this as any).attrMax, (this as any).dotsOverFive ? 10 : 5);
		},
		maxWillpower() {
			return (this as any).character.attributes.resolve + (this as any).character.attributes.composure;
		},
		// defense() {
		// 	return Math.min((this as any).character.attributes.dexterity, (this as any).character.attributes.wits) + ((this as any).character.skills.athletics || 0);
		// },
		initative() {
			return (this as any).character.attributes.dexterity + (this as any).character.attributes.composure;
		},
		maxFuel() {
			const character: Character = this.character;
			return character.power === 0 ? (this as any).character.splat === EnumSplat.VAMPIRE ? (this as any).attributes.stamina : 0 : character.power >= 5 ? character.power >= 9 ? character.power === 10 ? 75 : (50) : (10 + (character.power-4)*5) : (10 + character.power - 1);
		},
		perception() {
			return (this as any).character.attributes.resolve+(this as any).character.attributes.composure+((this as any).currentForm.perceptionMod || 0);
		},
		// abilities(): {[index: string]: Ability} {
		// 	const character: Character = this.character;
		// 	const obj: {[index: string]: Ability} = {};
		// 	// character.abilityArr.forEach(el => {
		// 	// 	obj[el.name.toLowerCase()] = {level: el.level, name: el.name};
		// 	// });
		// 	return obj;
		// },
		currentForm(): Form {
			return this.character.currentFormObj;
		},
		rotes() {
			const rotes = [...(this as any).character.rotes];
			rotes.push({});
			return rotes;
		}
	},
	methods: {
		async rollTest() {
			const results = [] as string[][];

			const maxTarget = 12;
			const maxDice = 15;

			const rollIterations = 5;

			for (let dice = 1; dice <= maxDice; dice++) {
				for (let target = 1; target <= maxTarget; target++) {

					// const succArr = [];
					// for (let i = 0; i < 10; i++) {
					let succs = 0;
					for (let j = 0; j < rollIterations; j++) {
						const suc = await this.roll(dice, {} as any);
						
						if (suc >= target) {
							succs++;
						}
					}
					// succArr.push(succs/target);
					// }
					if (!results[dice-1]) results[dice-1] = [];
					// results[dice-1][target-1] = succArr.reduce((prev, val) => prev+val) / 10;
					results[dice-1][target-1] = ((succs/rollIterations/target)*100).toFixed(2);
				}
			}
			return results;
		},
		// getRandom(num: number, min: number, max: number): number[] {

		// 	if (window.crypto && window.crypto.getRandomValues) {
		// 		const randomIntArray = function (length, min, max) {
		// 			return new Array(length).fill(0).map(() => randomInt(min, max))
		// 		}

		// 		return Array.from();
		// 	} else {
		// 		return Array.from({length: num}, () => Math.round(Math.random() * max-1)+min);
		// 	}
		// },
		rollSelected(opts: any) {
			const vals = Object.values(this.selectedTraits) as any[];
			if (vals.length === 0) return 0;

			const dice = vals.reduce((prev,val) => prev+val);

			const result = this.roll(dice, opts);

			alert(`Rolled ${dice} dice, got ${result} successes`);
		},
		roll(dice: number, opts: {
			again: number;
			bonus: number;
			treshold: number;
			values: boolean;
		}): number[] | number {
			opts = opts || {} as any;
			opts.again    = opts.again || 10;
			opts.bonus    = opts.bonus || 0;
			opts.treshold = opts.treshold || 8;
			opts.values   = opts.values || false;

			// const self = this;

			if (dice <= 0) {
				dice = 1;
				opts.again = 11;
			}

			let result = RandomUtil.randomIntArray(dice, 1, 10);
			// let result = await RandomUtil.generateIntegerSequence(dice, 1, 10);

			const again = result.filter(num => num >= opts.again).length;

			if (again > 0 && again !== 1) {
				result = result.concat(this.roll(again, Object.assign({}, opts, {values: true})));
			}

			// .flatMap(num => num >= opts.again ? [num, ...this.roll(1, Object.assign({}, opts, {values: true})) as number[]] : [num]) as number[];


			const succs = result.filter(el => el >= opts.treshold).length;

			return opts.values ? result : succs+opts.bonus;
		},
		selectTrait(name: string, opts: {attr?: boolean; skill?: boolean; skillCat: string; ability?: boolean}) {
			if (this.selectedTraits[name] !== undefined) {
				delete this.selectedTraits[name];
			} else if (name) {
				if (Object.keys(this.selectedTraits).length === 3) {
					this.selectedTraits = {};
				}

				// eslint-disable-next-line @typescript-eslint/no-this-alias
				const self = this;
				console.log(name);

				this.selectedTraits[name] = computed(function() {
					const attributes = self.character.attributes;
					const skills = self.character.skills;
					const abilities = self.character.abilities;
					const merits = self.character.merits;
					const obj = (opts.attr ? attributes : opts.skill ? skills : opts.ability ? abilities[name] ? abilities : merits : self.character);

					const res = (opts.ability ? obj[name].level : obj[name]) || 0;
					return res+(res === 0 && opts.skill ? Object.values(self.skillCats)[opts.skillCat as any] : 0);
				});
				console.log(this.selectedTraits);
			}
		},
		setAttr(attr: string, val: number) {
			const character: Character = (this as any).character;

			// if (!character.attributes)
			// 	(character.attributes as any) = {};

			(character.baseAttributes as any)[attr] = ((character.attributes as any)[attr] === val && val !== 1) ? val-1 : val;
		},
		setSkill(attr: string, val: number) {
			const character: Character = (this as any).character;

			if (!character.skills)
				character.skills = {};


			// console.log(character.skills);

			character.skills[attr] = (character.skills[attr] === val) ? val-1 : val;
			// console.log(character.skills);

		},
		setTrait(trait: string, val: number, opts: {off?: number; min?: number}) {
			const character: any = (this as any).character;

			// eslint-disable-next-line prefer-const
			let {off, min} = opts || {};

			off = off || 1;

			character[trait] = (character[trait] === val) ? val-off : val;
			
			// console.log(character[trait], min);
			character[trait] = Math.max(min || 0, character[trait]);

			if (trait === "beats" || trait === "alternateBeats") {
				if (character[trait] === 5) {
					character[trait] = 0;

					trait === "beats" ? character["experience"]++ : character["alternateExperience"]++;
				}
			}
		},
		doInputRote(ability: any, i: number) {
			if (!this.character.rotes[i]) {
				this.character.rotes[i] = ability;
			}
			// console.log(ability);
			if (!ability.arcanum &&
				!ability.spell  &&
				!ability.creator &&
				!ability.roteSkill
			) {
				// console.log(ability);
				// eslint-disable-next-line vue/no-mutating-props
				this.character.rotes.splice(i, 1);
			}
		},
		formDefense(form: Form) {
			return Math.min((this as any).character.attributes.dexterity - (this as any).currentForm.dexterityMod + form.dexterityMod, (this as any).character.attributes.wits) + ((this as any).character.skills.athletics || 0);
			// return (this as any).character.defense - (this as any).currentForm.dexterityMod + form.dexterityMod;
		},
		specialtyDropDown(skill: string) {
			if (this.specialtyDropSelect === skill) {
				if (this.character.specialties[skill] && this.character.specialties[skill].length === 0) {
					this.character.specialties[skill] = undefined;
				}

				this.specialtyDropSelect = null;
			} else {
				if (!this.character.specialties[skill]) {
					this.character.specialties[skill] = [];
				}

				this.specialtyDropSelect = skill;
			}
		}
	},
	data() {
		return {
			characters: null as any,
			character: null as any,
			sizeMinusForm: null as any,

			dotsOverFive: false,
			specialtyDropSelect: null as string | null,

			selectedTraits: {} as {[index: string]: Ref<number>},

			EnumSplat, ref, RandomUtil,

			attributeNames: [
				["intelligence", "wits"        , "resolve"],
				["strength"    , "dexterity"   , "stamina"],
				["presence"    , "manipulation", "composure"],
			],
			skills: [
				["academics", "computer", "crafts", "investigation", "medicine", "occult", "politics", "science"],
				["athletics", "brawl", "drive", "firearms", "larceny", "stealth", "survival", "weaponry"],
				["animal_ken", "empathy", "expression", "intimidation", "persuasion", "socialize", "streetwise", "subterfuge"],
			],
			skillCats: {"mental": -3, "physical": -1, "social": -1} as {[index: string]: number}
		};
	},
	beforeMount() {
		(window as any).vue = this;
		this.characters = JSON.parse(localStorage.characters) || {};

		this.character = this.characters[this.id];
		// const blankChara = new Character({splat: this.character.splat});

		// this.character = Object.assign({}, blankChara, this.character);

		// this.character = toRefs(new Character(this.character));
		this.character = new Character(this.character);

		// this.character.abilityArr = Object.values(this.abilities);

		// eslint-disable-next-line @typescript-eslint/no-this-alias
		const self = this;
		this.sizeMinusForm = computed({
			get() {
				return self.character.size - self.currentForm.sizeMod;	
			},
			set(val) {
				self.character.size = val;
			}
		});

		this.characters[this.id] = this.character;
	},
	watch: {
		characters: {
			handler(newVal, oldVal) {
				// (this as any).characters[(this as any).id] = newVal;
				localStorage.characters = JSON.stringify(newVal);
			},
			deep: true,
		}
		// attrMax: function(newVal, oldVal) {
		// 	if (newVal > 5) {
		// 		$(".sheet-dot").attr("11");
		// 	} 
		// }
		// $route: function(newRoute): any {
		// }
	},
});

export default x;
</script>

<style lang="scss">

$xs-max: 575.98px;
$sm-min: 576px;
$sm-max: 767.98px;

header {
	margin-top: 0px;
	min-height: 35px;
	text-align: left;
	// background-color: $s-dark;
	// border-bottom: 1px solid;

	width: 100%;

	$spread: -10px;

	-webkit-box-shadow: 0px 6px 17px $spread rgba(0, 0, 0, 0.34);
	box-shadow: 0px 6px 17px $spread rgba(0, 0, 0, 0.34);

	div {
		margin-left: 15px;
		padding-top: 5px;
		margin-bottom: 5px;
		// shadow
	}
}

input {
	background-color: transparent;
	border: none;
	color: inherit;
}

input:focus {
	outline: none;
}

.bar {
	white-space: nowrap;
	margin-top: 15px;
}

.block {
	text-align: left;
	overflow: hidden;

	width: 100%;

	margin-bottom: 15px;

	@media (max-width: $xs-max) {
		margin: auto;
		margin-top: 15px;
		width: 60%;
	}



}

.block.col-sm-2 {
	@media (max-width: $xs-max) {
			display: none;
	}
}

.separator {
	text-transform: uppercase;
	text-align: center;

	margin-top: 0px;
	// margin-bottom: 15px;
}

.block .sheet-dots {
	float: right;
}

button.sheet-dot {
	-moz-border-radius: 50%;
    -webkit-border-radius: 50%;
    border-radius: 50%;
}

button.sheet-dot-full {
	background: $accent !important;
}

.sheet-dot,
.sheet-box {
	$radius: 15px;

    width: $radius;
    height: $radius;

    padding: 0;
    border: solid 1px #000000;
    line-height: 11px;
    background-color: #efefef;

	margin-right: 1px;
}

#skills .sheet-dot {
	@media (min-width: $sm-min) and (max-width: $sm-max) {
		$radius: 11px;

		width: $radius;
		height: $radius;
	}
	@media (max-width: $xs-max) {
		$radius: 11px;

		width: $radius;
		height: $radius;
	}
}

#skills .sheet-dot-small {
	@media (min-width: $sm-min) and (max-width: $sm-max) {
		$radius: 8px;

		width: $radius;
		height: $radius;
	}
	@media (max-width: $xs-max) {
		$radius: 5px;

		width: $radius;
		height: $radius;
	}
}

#attrBar .sheet-dot {
	@media (min-width: $sm-min) and (max-width: $sm-max) {
		$radius: 11px;

		width: $radius;
		height: $radius;
	}
	@media (max-width: $xs-max) {
		$radius: 11px;

		width: $radius;
		height: $radius;
	}
}

#attrBar .sheet-dot-small {
	@media (min-width: $sm-min) and (max-width: $sm-max) {
		$radius: 6px;

		width: $radius;
		height: $radius;
	}
	@media (max-width: $xs-max) {
		$radius: 5px;

		width: $radius;
		height: $radius;
	}
}

button.sheet-dot-small {
	$radius: 12px;

	width: $radius;
	height: $radius;
}

#minorTraits {
	font-size: 14.2pt;
}

.subtitle {
	margin-top:-13px;
	// margin-bottom:-10px;
	display:block;
	text-align:center;

	font-size: 8pt;

	color: black;
}

.fab {
	position: fixed;

	right: 10px;
	bottom: 10px;
}

#rotes {
	text-align: center;
	#rote-level input {
		width: 220%;
	}
	input {
		width: 100%;
		margin-left: 10px;
	}

}

.attr-input {
	border: 1px solid black;
	border-radius: 10px;
	background-color: #484747;
	color: white;
}

#werewolf-forms {
	// background-image: url('../assets/images/werewolf-forms.webp')
}
.dropdown-toggle {
	border: none;

	height: 100%;
	// padding-bottom: -10px;

	vertical-align: center;
	text-align: center;

	span {
		// : 10px;
		font-size: 14pt;
	}
}

.dropdown-toggle::after {
	display: none !important;
	// padding-top: 10px;
}

.specialties {
	$color: green;

	color: green;
	// fill: black;
    // stroke: red;
    // stroke-width: 3;
// text-shadow: 2px 0 0 $color, -2px 0 0 $color, 0 2px 0 $color, 0 -2px 0 $color, 1px 1px $color, -1px -1px 0 $color, 1px -1px 0 $color, -1px 1px 0 $color;
}

</style>