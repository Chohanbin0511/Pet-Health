<template>
	<v-btn>
		<!-- <v-btn @click="createFamily()"> 가족 생성</v-btn> -->
		<v-row>
			<v-dialog v-model="dialog">
				<template v-slot:activator="{ props }">
					<v-btn color="" v-bind="props">가족 그룹 생성</v-btn>
				</template>
				<v-card>
					<v-card-title class="mt-5">
						<span class="text-h6">가족 그룹 정보</span>
					</v-card-title>
					<v-card-text>
						<v-container>
							<v-row>
								<v-col cols="12">
									<v-text-field
										v-model="inputData.name"
										label="*그룹명"
										required
									></v-text-field>
								</v-col>
							</v-row>
						</v-container>
						<small>*필수 입력 사항</small>
					</v-card-text>
					<v-card-actions>
						<v-spacer></v-spacer>
						<v-btn
							color="blue-darken-1"
							variant="text"
							@click="closeInsertForm"
						>
							Close
						</v-btn>
						<v-btn
							color="blue-darken-1"
							variant="text"
							@click="fetchCreateGroup"
						>
							Save
						</v-btn>
					</v-card-actions>
				</v-card>
			</v-dialog>
		</v-row>
	</v-btn>
</template>

<script setup>
import { ref, inject } from 'vue';
import { useAuthStore } from '@/stores/auth';
import { createGroup, getGroupList } from '@/api/fhirApi';

const progress = inject('nprogress');
const rootMethods = inject('rootMethods');

const dialog = ref(false);

const emit = defineEmits(['update:groupList']);

// 가족 생성
const inputData = ref({
	name: null,
});
const store = useAuthStore();
const userInfo = useAuthStore().userInfo;
const { SET_GROUPLIST } = store;

// 가족 Group 생성
const fetchCreateGroup = async () => {
	if (!inputData.value.name) {
		return false;
	}

	const resource = {
		resourceType: 'Group',
		identifier: [
			{
				system: 'https://redwoodplatformtest-chohbin.netlify.app',
				value: userInfo.patientId,
			},
		],
		active: true,
		type: 'animal',
		actual: true,
		name: inputData.value.name,
		quantity: 0,
		member: [],
	};

	try {
		progress.value = 0.1;
		const response = await createGroup(resource);
		console.log('response', response);
		rootMethods.openToast('그룹 생성이 완료되었습니다.');
		fetchGroupList();
		emit('update:groupList');
		dialog.value = false;
	} catch (error) {
		console.error(error);
	} finally {
		progress.value = 1;
	}
};

const fetchGroupList = async () => {
	// store
	const {
		data: { total },
		data: { entry },
	} = await getGroupList(userInfo.patientId);
	if (total > 0) {
		const {
			resource: { name },
			resource: { member },
		} = entry[0];
		SET_GROUPLIST(total, name, member);
	}
};

const closeInsertForm = () => {
	dialog.value = false;
	inputData.value.name = null;
};
</script>

<style lang="scss" scoped></style>
