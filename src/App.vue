<!-- App.vue -->
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form';
import { zodValidator } from '@tanstack/zod-form-adapter';
import { z } from 'zod';
import FieldInfo from './components/FieldInfo.vue';

const userSchema = z.object({
    name: z.string().min(3, 'Name must be at least 3 characters'),
    hobbies: z.array(
        z.object({
            name: z.string().min(3, 'Hobby name must be at least 3 characters'),
            description: z.string(),
        })
    )
});

type User = z.infer<typeof userSchema>;

const form = useForm({
    defaultValues: {
        name: '',
        hobbies: [],
    } as User,
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
                        .min(3, 'Name must be at least 3 characters'),
                    onChangeAsyncDebounceMs: 500,
                    onChangeAsync: onChangeName,
                }"
            >
                <template v-slot="{ field, state }">
                    <label
                        :for="field.name"
                        class="form-label"
                    >Name</label>
                    <input
                        class="form-control"
                        :aria-describedby="field.name + '-help'"
                        :id="field.name"
                        :name="field.name"
                        :value="field.state.value"
                        @blur="field.handleBlur"
                        @input="e => field.handleChange((e.target as HTMLInputElement).value)"
                    />
                    <FieldInfo :state="state" />
                    <div
                        :id="field.name + '-help'"
                        class="form-text"
                    >Must not contain the word "error"</div>
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
                                                <label
                                                    :for="field.name"
                                                    class="form-label"
                                                >Name</label>
                                                <input
                                                    class="form-control"
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
                                                <label
                                                    :for="field.name"
                                                    class="form-label"
                                                >Description</label>
                                                <input
                                                    class="form-control"
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
