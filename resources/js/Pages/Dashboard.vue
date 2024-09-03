<script setup>
import { ref } from 'vue';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Head, Link, router } from '@inertiajs/vue3';

const props = defineProps(['games']);

const games = ref(props.games.data);

Echo.private('lobby')
    .listen('GameJoined', (event) => {
        games.value = games.value.filter((game) => game.id !== event.game.id);

        if(games.value.length < 5) {
            router.reload({ only: ['games'], onSuccess: () => games.value = props.games.data })
        }
    })
    .listen('GameCreated', (event) => {
        games.value.push(event.game);
        
        router.reload({ only: ['games'], onSuccess: () => games.value = props.games.data });
    });
</script>

<template>
    <Head title="Dashboard" />

    <AuthenticatedLayout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">Dashboard</h2>
        </template>

        <div class="py-12">
            <div class="max-w-4xl mx-auto sm:px-6 lg:px-8">
                <Link :href="route('games.store')" method="post" as="button" class="py-4 px-6 inline-flex items-center bg-gray-800 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-700 focus:bg-gray-700 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150">Create Game</Link>
            
                <ul class="divide-y-2 mt-8">
                    <li v-for="game in games" :key="game.id" class="py-6 flex justify-between px-4 items-center">
                        <span>{{ game.player_one.name }}</span>
                        <Link :href="route('games.join', game)" method="post" as="button" class="text-white bg-slate-500 hover:bg-slate-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800">Join Game</Link>
                    </li>
                </ul>
            </div>
        </div>

    </AuthenticatedLayout>
</template>
