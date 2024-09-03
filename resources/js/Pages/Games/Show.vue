<script setup>
import { computed, ref } from 'vue';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Toaster, toast } from 'vue-sonner';

const board = ref([0, 0, 0, 0, 0, 0, 0, 0, 0]);

const firstTurn = computed(() => board.value.reduce(
    (acumulator, current_value) => acumulator + current_value, 0)
);

const fillSquare = (index) => {
    board.value[index] = firstTurn.value ? -1 : 1;

    const winningLine = lines.map((line) => line.reduce((acumulator, index) => acumulator + board.value[index], 0))
        .find((sum) => Math.abs(sum) === 3);

    if (winningLine === 3) {
        toast.success('O won!'); 
        return;
    }

    if (winningLine === -3) {
        toast.success('X won!');
        return;
    }

    if (! board.value.includes((0))) {
        toast('Draw!');
    }
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
    </AuthenticatedLayout>
</template>