<script setup lang="ts">
import Mail from "@/Types/Mail";
import { Card, Tag, Button } from "primevue";
import { PropType } from "vue";
import { Link } from "@inertiajs/vue3";
import Toast from "primevue/toast";
import { useToast } from "primevue/usetoast";
import dayjs from "dayjs";

const toast = useToast();

// Define props for the email data
defineProps({
    mail: {
        type: Object as PropType<Mail>,
        required: true,
    },
});

const confirmResend = (event: MouseEvent) => {
    if (!confirm('Are you sure you want to resend this email?')) {
        event.preventDefault();
    }
};
</script>

<template>
    <Toast />

    <div class="h-full p-2">
        <Card
            class="min-h-full"
            :pt="{
                body: {
                    class: 'grow',
                },
                content: {
                    class: 'grow  flex items-stretch',
                },
            }"
        >
            <template #content>
                <div class="flex w-full flex-col space-y-4">
                    <!-- Header Information -->
                    <div class="flex-none">
                        <div class="flex items-baseline justify-between">
                            <h1 class="mb-4 text-2xl font-bold">{{ mail.subject }}</h1>
                            <div class="text-muted-color text-sm">
                                {{ dayjs(mail.sent_at).format("MMMM D, YYYY h:mm A") }}
                            </div>
                        </div>
                        <div class="flex items-center justify-between">
                            <div class="">
                                <p class="">
                                    To: {{ mail.to_name ? `${mail.to_name} <${mail.to_email}>` : mail.to_email }}
                                </p>
                                <p class="text-muted-color text-sm">
                                    From:
                                    {{ mail.from_name ? `${mail.from_name} <${mail.from_email}>` : mail.from_email }}
                                </p>
                            </div>
                            <div class="space-x-2">
                                <Tag
                                    :severity="mail.status === 'success' ? 'success' : 'danger'"
                                    :value="mail.status"
                                    rounded
                                    class="align-middle"
                                />
                                <!-- Re-send email -->
                                <Link
                                    :href="route('mail.resend', { id: mail.id })"
                                    method="post"
                                    as="button"
                                    @click="confirmResend"
                                    :onSuccess="() => toast.add({
                                        severity: 'success',
                                        summary: 'Email re-sent',
                                        detail: 'The email was successfully re-sent.',
                                        life: 5000
                                    })"
                                    :onError="() => toast.add({
                                        severity: 'error',
                                        summary: 'Resend failed',
                                        detail: 'The email could not be re-sent.',
                                        life: 5000
                                    })"
                                >
                                    <Button
                                        label="Resend"
                                    />
                                </Link>
                            </div>
                        </div>
                    </div>

                    <!-- Error Message (if any) -->
                    <div
                        v-if="mail.error_message"
                        class="flex-none rounded border border-red-200 bg-red-50 p-4"
                    >
                        <p class="text-red-800">{{ mail.error_message }}</p>
                    </div>

                    <!-- Email Body -->
                    <div class="mt-4 flex grow items-stretch overflow-clip rounded-lg">
                        <iframe
                            class="w-full"
                            :srcdoc="mail.content_html"
                            frameborder="0"
                            allowfullscreen
                        ></iframe>
                    </div>
                </div>
            </template>
        </Card>
    </div>
</template>
