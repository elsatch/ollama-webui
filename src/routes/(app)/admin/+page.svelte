<script>
	import { WEBUI_API_BASE_URL } from '$lib/constants';
	import { config, user } from '$lib/stores';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	import toast from 'svelte-french-toast';

	import { updateUserRole, getUsers, deleteUserById } from '$lib/apis/users';
	import { getSignUpEnabledStatus, toggleSignUpEnabledStatus } from '$lib/apis/auths';
	import EditUserModal from '$lib/components/admin/EditUserModal.svelte';
	import SettingsModal from '$lib/components/admin/SettingsModal.svelte';

	let loaded = false;
	let users = [];

	let selectedUser = null;

	let showSettingsModal = false;
	let showEditUserModal = false;

	const updateRoleHandler = async (id, role) => {
		const res = await updateUserRole(localStorage.token, id, role).catch((error) => {
			toast.error(error);
			return null;
		});

		if (res) {
			users = await getUsers(localStorage.token);
		}
	};

	const editUserPasswordHandler = async (id, password) => {
		const res = await deleteUserById(localStorage.token, id).catch((error) => {
			toast.error(error);
			return null;
		});
		if (res) {
			users = await getUsers(localStorage.token);
			toast.success('Successfully updated');
		}
	};

	const deleteUserHandler = async (id) => {
		const res = await deleteUserById(localStorage.token, id).catch((error) => {
			toast.error(error);
			return null;
		});
		if (res) {
			users = await getUsers(localStorage.token);
		}
	};

	onMount(async () => {
		if ($user?.role !== 'admin') {
			await goto('/');
		} else {
			users = await getUsers(localStorage.token);
		}
		loaded = true;
	});
</script>

{#key selectedUser}
	<EditUserModal
		bind:show={showEditUserModal}
		{selectedUser}
		sessionUser={$user}
		on:save={async () => {
			users = await getUsers(localStorage.token);
		}}
	/>
{/key}

<SettingsModal bind:show={showSettingsModal} />

<div
	class=" bg-white dark:bg-gray-900 dark:text-gray-100 min-h-screen w-full flex justify-center font-mona"
>
	{#if loaded}
		<div class="w-full max-w-3xl px-10 md:px-16 min-h-screen flex flex-col">
			<div class="py-10 w-full">
				<div class=" flex flex-col justify-center">
					<div class=" flex justify-between items-center">
						<div class=" text-2xl font-semibold">Users ({users.length})</div>
						<div>
							<button
								class="flex items-center space-x-1 border border-gray-200 dark:border-gray-600 px-3 py-1 rounded-lg"
								type="button"
								on:click={() => {
									showSettingsModal = !showSettingsModal;
								}}
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									viewBox="0 0 16 16"
									fill="currentColor"
									class="w-4 h-4"
								>
									<path
										fill-rule="evenodd"
										d="M6.955 1.45A.5.5 0 0 1 7.452 1h1.096a.5.5 0 0 1 .497.45l.17 1.699c.484.12.94.312 1.356.562l1.321-1.081a.5.5 0 0 1 .67.033l.774.775a.5.5 0 0 1 .034.67l-1.08 1.32c.25.417.44.873.561 1.357l1.699.17a.5.5 0 0 1 .45.497v1.096a.5.5 0 0 1-.45.497l-1.699.17c-.12.484-.312.94-.562 1.356l1.082 1.322a.5.5 0 0 1-.034.67l-.774.774a.5.5 0 0 1-.67.033l-1.322-1.08c-.416.25-.872.44-1.356.561l-.17 1.699a.5.5 0 0 1-.497.45H7.452a.5.5 0 0 1-.497-.45l-.17-1.699a4.973 4.973 0 0 1-1.356-.562L4.108 13.37a.5.5 0 0 1-.67-.033l-.774-.775a.5.5 0 0 1-.034-.67l1.08-1.32a4.971 4.971 0 0 1-.561-1.357l-1.699-.17A.5.5 0 0 1 1 8.548V7.452a.5.5 0 0 1 .45-.497l1.699-.17c.12-.484.312-.94.562-1.356L2.629 4.107a.5.5 0 0 1 .034-.67l.774-.774a.5.5 0 0 1 .67-.033L5.43 3.71a4.97 4.97 0 0 1 1.356-.561l.17-1.699ZM6 8c0 .538.212 1.026.558 1.385l.057.057a2 2 0 0 0 2.828-2.828l-.058-.056A2 2 0 0 0 6 8Z"
										clip-rule="evenodd"
									/>
								</svg>

								<div class=" text-xs">Admin Settings</div>
							</button>
						</div>
					</div>
					<div class=" text-gray-500 text-xs font-medium mt-1">
						Click on the user role cell in the table to change a user's role.
					</div>

					<hr class=" my-3 dark:border-gray-600" />

					<div class="scrollbar-hidden relative overflow-x-auto whitespace-nowrap">
						<table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
							<thead
								class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
							>
								<tr>
									<th scope="col" class="px-6 py-3"> Name </th>
									<th scope="col" class="px-6 py-3"> Email </th>
									<th scope="col" class="px-6 py-3"> Role </th>
									<th scope="col" class="px-6 py-3"> Action </th>
								</tr>
							</thead>
							<tbody>
								{#each users as user}
									<tr class="bg-white border-b dark:bg-gray-900 dark:border-gray-700">
										<th
											scope="row"
											class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white w-fit"
										>
											<div class="flex flex-row">
												<img
													class=" rounded-full max-w-[30px] max-h-[30px] object-cover mr-4"
													src={user.profile_image_url}
													alt="user"
												/>

												<div class=" font-semibold self-center">{user.name}</div>
											</div>
										</th>
										<td class="px-6 py-4"> {user.email} </td>
										<td class="px-6 py-4">
											<button
												class="  dark:text-white underline"
												on:click={() => {
													if (user.role === 'user') {
														updateRoleHandler(user.id, 'admin');
													} else if (user.role === 'pending') {
														updateRoleHandler(user.id, 'user');
													} else {
														updateRoleHandler(user.id, 'pending');
													}
												}}>{user.role}</button
											>
										</td>
										<td class="px-6 py-4 space-x-1 text-center flex justify-center">
											<button
												class="self-center w-fit text-sm p-1.5 border dark:border-gray-600 rounded-xl flex"
												on:click={async () => {
													showEditUserModal = !showEditUserModal;
													selectedUser = user;
												}}
											>
												<svg
													xmlns="http://www.w3.org/2000/svg"
													viewBox="0 0 16 16"
													fill="currentColor"
													class="w-4 h-4"
												>
													<path
														fill-rule="evenodd"
														d="M11.013 2.513a1.75 1.75 0 0 1 2.475 2.474L6.226 12.25a2.751 2.751 0 0 1-.892.596l-2.047.848a.75.75 0 0 1-.98-.98l.848-2.047a2.75 2.75 0 0 1 .596-.892l7.262-7.261Z"
														clip-rule="evenodd"
													/>
												</svg>
											</button>

											<button
												class="self-center w-fit text-sm p-1.5 border dark:border-gray-600 rounded-xl flex"
												on:click={async () => {
													deleteUserHandler(user.id);
												}}
											>
												<svg
													xmlns="http://www.w3.org/2000/svg"
													fill="none"
													viewBox="0 0 24 24"
													stroke-width="1.5"
													stroke="currentColor"
													class="w-4 h-4"
												>
													<path
														stroke-linecap="round"
														stroke-linejoin="round"
														d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0"
													/>
												</svg>
											</button>
										</td>
									</tr>
								{/each}
							</tbody>
						</table>
					</div>
				</div>
			</div>
		</div>
	{/if}
</div>

<style>
	.font-mona {
		font-family: 'Mona Sans';
	}

	.scrollbar-hidden::-webkit-scrollbar {
		display: none; /* for Chrome, Safari and Opera */
	}

	.scrollbar-hidden {
		-ms-overflow-style: none; /* IE and Edge */
		scrollbar-width: none; /* Firefox */
	}
</style>
