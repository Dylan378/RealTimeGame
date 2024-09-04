<script setup>
import { computed, ref, onUnmounted } from 'vue';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Toaster, toast } from 'vue-sonner';
import { router } from '@inertiajs/vue3';

const props = defineProps(['game']);

const board = ref(props.game.state ?? [0, 0, 0, 0, 0, 0, 0, 0, 0]);

const players = ref();

const firstTurn = computed(() => board.value.reduce(
    (acumulator, current_value) => acumulator + current_value, 0)
);

const fillSquare = (index) => {
    board.value[index] = firstTurn.value ? -1 : 1;

    router.put(route('games.update', props.game.id), {
        state: board.value,
    });

    const winningLine = lines.map((line) => line.reduce((acumulator, index) => acumulator + board.value[index], 0))
        .find((sum) => Math.abs(sum) === 3);

    if (winningLine === 3) {
        toast.success('O won!', {
            action: {
                label: 'Reset game',
                onClick: () => resetGame()
            }
        }); 
        return;
    }

    if (winningLine === -3) {
        toast.success('X won!', {
            action: {
                label: 'Reset game',
                onClick: () => resetGame()
            }
        }); 
        return;
    }

    if (! board.value.includes((0))) {
        toast('Draw!', {
            action: {
                label: 'Reset game',
                onClick: () => resetGame()
            }
        });
    }

    onUnmounted(() => {
        Echo.leave(`games.${props.game.id}`);
    });
}

const lines = [
    [0, 1, 2],
    [3, 4 ,5],
    [6, 7, 8],

    [0, 3, 6],
    [1, 4 ,7],
    [2, 5, 8],
    
    [0, 4, 8],
    [2, 4 ,6],
]

const resetGame = () => {
    board.value = [0, 0, 0, 0, 0, 0, 0, 0, 0];
}

Echo.join(`games.${props.game.id}`)
    .here((users) => players.value = users)
    .joining((user) => {
        router.reload({
            onSuccess: () => players.value.push(user)
        });
        toast('Player 2 has joined the game');
    })
    .leaving((user) => {
        router.reload({
            onSuccess: () => {
                players.value = players.value.filter(({ id }) => id !== user.id);
            }   
        });
    })
    .listen('PlayerMadeMove', ({game}) => {
        board.value = game.state;
    });

</script>

<template>
    <AuthenticatedLayout class="">
        <Toaster closeButton rich-colors position="top-center" />
        <menu class="grid grid-cols-3 gap-1.5 w-0 min-w-fit mx-auto mt-12">
            <li v-for="(square, index) in board" class="size-32 grid place-items-center bg-slate-300 rounded-md">
                <button @click="fillSquare(index)" v-if="square === 0" class="place-self-stretch bg-stone-200 hover:bg-slate-300"></button>
                <span v-else v-text="square === -1 ? 'X' : 'O' " class="text-5xl font-extrabold"></span>
            </li>
        </menu>

        <ul class="max-w-sm mx-auto my-6">
            <li class="my-4 flex items-center gap-3">
                <span class="px-1.5 font-bold rounded bg-slate-400">X</span>
                <span>{{ game.player_one.name }}</span>
                <span class="relative flex h-2 w-2">
                    <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-sky-400 opacity-75"></span>
                    <span class="relative inline-flex rounded-full h-2 w-2 bg-sky-500"></span>
                </span>
            </li>

            <li v-if="game.player_two" class="my-4 flex items-center gap-3">
                <span class="px-1.5 font-bold rounded bg-slate-400">O</span>
                <span>{{ game.player_two.name }}</span>
                <span class="relative flex h-2 w-2">
                    <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-sky-400 opacity-75"></span>
                    <span class="relative inline-flex rounded-full h-2 w-2 bg-sky-500"></span>
                </span>
            </li>
            
            <li v-else class="flex items-center gap-3">
                <svg aria-hidden="true" class="w-5 h-5 text-gray-200 animate-spin fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
                    <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
                    <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
                </svg>
                <span class="">Waiting for player 2 ...</span>
            </li>

        </ul>
    </AuthenticatedLayout>
</template>