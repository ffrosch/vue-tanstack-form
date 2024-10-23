<!-- App.vue -->
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form';
import { zodValidator } from '@tanstack/zod-form-adapter';
import { z } from 'zod';
import FieldInfo from './components/FieldInfo.vue';

const form = useForm({
    defaultValues: {
        name: '',
        hobbies: [] as Array<{ name: string, description: string; }>,
    },
    onSubmit: async ({ value }) => {
        // Do something with form data
        console.log(value);
    },
    validatorAdapter: zodValidator(),
});

const name = form.useStore((state) => state.values.name);
const hobbies = form.useStore((state) => state.values.hobbies);

const onChangeName = z.string().refine(
    async (value) => {
        await new Promise((resolve) => setTimeout(resolve, 1000));
        return !value.includes('error');
    },
    {
        message: 'Name must not contain error',
    }
);

const buttonClass = "bg-violet-500 hover:bg-violet-600 focus:outline-none focus:ring focus:ring-violet-300 active:bg-violet-700 px-5 py-2 text-sm leading-5 rounded-full font-semibold text-white";
</script>

<template>
    <form @submit.prevent.stop="form.handleSubmit">
        <div class="grid w-25">
            <h2 class="text-3xl">Personal Information</h2>
            <form.Field
                name="name"
                :validators="{
                    onChange: z
                        .string()
                        .min(3, 'First name must be at least 3 characters'),
                    onChangeAsyncDebounceMs: 500,
                    onChangeAsync: onChangeName,
                }"
            >
                <template v-slot="{ field, state }">
                    <label :htmlFor="field.name">Name (must not contain `error`): </label>
                    <input
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="e => field.handleChange((e.target as HTMLInputElement).value)"
                    />
                    <FieldInfo :state="state" />
                    {{ field.state.value }}
                </template>
            </form.Field>
            <form.Field
                name="hobbies"
                mode="array"
            >
                <template v-slot="{ field: hobbiesField }">
                    <div>
                        <h2 class="text-3xl">Hobbies</h2>
                        <div>
                            <div v-if="Array.isArray(hobbiesField.state.value) && !hobbiesField.state.value.length">
                                No hobbies found.
                            </div>
                            <div v-else>
                                <div
                                    class="position-relative"
                                    v-for="(_, i) in hobbiesField.state.value"
                                    :key="i"
                                >
                                    <button
                                        type="button"
                                        class="btn-close position-absolute end-0 me-3"
                                        @click="hobbiesField.removeValue(i)"
                                    >
                                    </button>
                                    <form.Field :name="`hobbies[${i}].name`">
                                        <template v-slot="{ field, state }">
                                            <div>
                                                <label :for="field.name">Name:</label>
                                                <input
                                                    :id="field.name"
                                                    :name="field.name"
                                                    :value="field.state.value"
                                                    @blur="field.handleBlur"
                                                    @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
                                                />
                                                <div class="text-red-500">
                                                    <FieldInfo :state="state" />
                                                </div>
                                            </div>
                                        </template>
                                    </form.Field>
                                    <form.Field :name="`hobbies[${i}].description`">
                                        <template v-slot="{ field, state }">
                                            <div>
                                                <label :for="field.name">Description:</label>
                                                <input
                                                    :id="field.name"
                                                    :name="field.name"
                                                    :value="field.state.value"
                                                    @blur="field.handleBlur"
                                                    @input="(e) => field.handleChange((e.target as HTMLInputElement).value)"
                                                />
                                                <div class="text-red-500">
                                                    <FieldInfo :state="state" />
                                                </div>
                                            </div>
                                        </template>
                                    </form.Field>
                                </div>
                            </div>
                            <button
                                type="button"
                                :class="buttonClass"
                                @click="
                                    hobbiesField.pushValue({
                                        name: '',
                                        description: '',
                                    })
                                    "
                            >
                                Add hobby
                            </button>
                        </div>
                    </div>
                </template>
            </form.Field>
        </div>
        <div>
            Hobbies: {{ hobbies }}
        </div>
        <button
            type="submit"
            class="btn btn-primary mt-3"
        >Submit</button>
    </form>
</template>
