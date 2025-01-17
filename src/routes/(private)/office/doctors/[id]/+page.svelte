<script lang="ts">
	import { useAuth } from '$lib/providers/auth-guard.svelte';
	import { LogOut } from 'svelte-feathers';
	import Navbar from './navbar.svelte';
	import type { PageData } from './$types';
	import { createQuery } from '@tanstack/svelte-query';
	import { OfficeService } from '$lib/services/office/office.service.svelte';
	import DoctorSchedule from '$lib/components/doctor/doctor-schedule.svelte';
	import AppointmentsTable from '$lib/components/doctor/appointments-table.svelte';
	import type { Appointment } from '$lib/services/office/interfaces/appointment';
	import AppointmentAddModal from './appointment-add-modal.svelte';

	const { data }: { data: PageData } = $props();

	const auth = useAuth();
	let isAddModalOpen = $state(false);

	let doctorQuery = createQuery({
		queryKey: ['/office/doctors/', data.id],
		queryFn: () => OfficeService.getDoctorInfo(data.id),
	});

	function closeAddModal() {
		isAddModalOpen = false;
		selectedAppointment = undefined;
	}

	let selectedAppointment: Appointment | undefined = $state();
	function openAddModal(Appointment?: Appointment) {
		selectedAppointment = Appointment;
		isAddModalOpen = true;
	}
	const { user } = useAuth();
</script>

<!-- HTML -->
<header
	class="sticky left-0 right-0 top-0 z-50 flex h-16 items-center justify-between bg-dark-main-100 px-5 text-2xl text-foreground-100"
>
	<div class="flex gap-4">
		<a href="/office">{user?.office.name}</a>
		<p>/</p>
		{#if $doctorQuery.data}
			<a href={`/office/doctors/${data.id}`}>
				{$doctorQuery.data?.name + ' ' + $doctorQuery.data?.last_name}
			</a>
		{/if}
	</div>
	<button onclick={auth.logout}>
		<LogOut class="size-5" />
	</button>
</header>

<div class="flex">
	<Navbar doctor={$doctorQuery.data} />

	<div class="flex flex-col items-center">
		<DoctorSchedule doctor_id={data.id} />
		<AppointmentsTable
			doctor_id={data.id}
			isDoctor={false}
			onAddAppointment={openAddModal}
		/>
		{#key selectedAppointment}
			<AppointmentAddModal
				appointment={selectedAppointment}
				isOpen={!!selectedAppointment || isAddModalOpen}
				onClose={closeAddModal}
				doctor_id={data.id}
			/>
		{/key}
	</div>
</div>
