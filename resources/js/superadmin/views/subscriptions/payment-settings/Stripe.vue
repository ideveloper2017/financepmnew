<template>
    <SuperAdminPageHeader>
        <template #header>
            <a-page-header :title="$t(`payment_settings.stripe_settings`)" class="p-0">
                <template #extra>
                    <a-button type="primary" @click="onSubmit">
                        <template #icon> <SaveOutlined /> </template>
                        {{ $t("common.update") }}
                    </a-button>
                </template>
            </a-page-header>
        </template>
        <template #breadcrumb>
            <a-breadcrumb separator="-" style="font-size: 12px">
                <a-breadcrumb-item>
                    <router-link :to="{ name: 'admin.dashboard.index' }">
                        {{ $t(`menu.dashboard`) }}
                    </router-link>
                </a-breadcrumb-item>
                <a-breadcrumb-item>
                    {{ $t("menu.payment_settings") }}
                </a-breadcrumb-item>
                <a-breadcrumb-item>
                    {{ $t("menu.stripe") }}
                </a-breadcrumb-item>
            </a-breadcrumb>
        </template>
    </SuperAdminPageHeader>

    <a-row>
        <a-col :xs="24" :sm="24" :md="24" :lg="4" :xl="4" class="bg-setting-sidebar">
            <SubscriptionSidebar />
        </a-col>
        <a-col :xs="24" :sm="24" :md="24" :lg="20" :xl="20">
            <admin-page-table-content>
                <a-card class="page-content-container mt-20 mb-20">
                    <a-form layout="vertical">
                        <a-row :gutter="16">
                            <a-col :xs="24" :sm="24" :md="12" :lg="12">
                                <a-form-item
                                    :label="$t('payment_settings.stripe_status')"
                                    name="stripe_status"
                                    :help="
                                        rules.stripe_status
                                            ? rules.stripe_status.message
                                            : null
                                    "
                                    :validateStatus="rules.stripe_status ? 'error' : null"
                                    class="required"
                                >
                                    <a-switch
                                        v-model:checked="formData.stripe_status"
                                        checkedValue="active"
                                        unCheckedValue="inactive"
                                    />
                                </a-form-item>
                            </a-col>
                        </a-row>

                        <template v-if="formData.stripe_status == 'active'">
                            <a-row :gutter="16">
                                <a-col :xs="24" :sm="24" :md="12" :lg="12">
                                    <a-form-item
                                        :label="$t('payment_settings.stripe_api_key')"
                                        name="stripe_api_key"
                                        :help="
                                            rules.stripe_api_key
                                                ? rules.stripe_api_key.message
                                                : null
                                        "
                                        :validateStatus="
                                            rules.stripe_api_key ? 'error' : null
                                        "
                                        class="required"
                                    >
                                        <a-input
                                            v-model:value="formData.stripe_api_key"
                                            :placeholder="
                                                $t('common.placeholder_default_text', [
                                                    $t('payment_settings.stripe_api_key'),
                                                ])
                                            "
                                        />
                                    </a-form-item>
                                </a-col>
                                <a-col :xs="24" :sm="24" :md="12" :lg="12">
                                    <a-form-item
                                        :label="$t('payment_settings.stripe_api_secret')"
                                        name="stripe_api_secret"
                                        :help="
                                            rules.stripe_api_secret
                                                ? rules.stripe_api_secret.message
                                                : null
                                        "
                                        :validateStatus="
                                            rules.stripe_api_secret ? 'error' : null
                                        "
                                        class="required"
                                    >
                                        <a-input
                                            v-model:value="formData.stripe_api_secret"
                                            :placeholder="
                                                $t('common.placeholder_default_text', [
                                                    $t(
                                                        'payment_settings.stripe_api_secret'
                                                    ),
                                                ])
                                            "
                                        />
                                    </a-form-item>
                                </a-col>
                            </a-row>

                            <a-row :gutter="16">
                                <a-col :xs="24" :sm="24" :md="12" :lg="12">
                                    <a-form-item
                                        :label="$t('payment_settings.stripe_webhook_key')"
                                        name="stripe_webhook_key"
                                        :help="
                                            rules.stripe_webhook_key
                                                ? rules.stripe_webhook_key.message
                                                : null
                                        "
                                        :validateStatus="
                                            rules.stripe_webhook_key ? 'error' : null
                                        "
                                        class="required"
                                    >
                                        <a-input
                                            v-model:value="formData.stripe_webhook_key"
                                            :placeholder="
                                                $t('common.placeholder_default_text', [
                                                    $t(
                                                        'payment_settings.stripe_webhook_key'
                                                    ),
                                                ])
                                            "
                                        />
                                    </a-form-item>
                                </a-col>
                                <a-col :xs="24" :sm="24" :md="12" :lg="12">
                                    <a-form-item
                                        :label="$t('payment_settings.webhook_url')"
                                        name="webhook_url"
                                    >
                                        <a-typography-text type="success">
                                            {{ webhookUrl }}
                                        </a-typography-text>
                                        <br />
                                        <small class="small-text-message">
                                            {{
                                                $t(
                                                    "payment_settings.stripe_webhook_message"
                                                )
                                            }}
                                        </small>
                                    </a-form-item>
                                </a-col>
                            </a-row>
                        </template>

                        <a-row :gutter="16">
                            <a-col :xs="24" :sm="24" :md="24" :lg="24">
                                <a-form-item>
                                    <a-button
                                        type="primary"
                                        @click="onSubmit"
                                        :loading="loading"
                                    >
                                        <template #icon> <SaveOutlined /> </template>
                                        {{ $t("common.update") }}
                                    </a-button>
                                </a-form-item>
                            </a-col>
                        </a-row>
                    </a-form>
                </a-card>
            </admin-page-table-content>
        </a-col>
    </a-row>
</template>
<script>
import { onMounted, ref } from "vue";
import {
    EyeOutlined,
    PlusOutlined,
    EditOutlined,
    DeleteOutlined,
    ExclamationCircleOutlined,
    SaveOutlined,
} from "@ant-design/icons-vue";
import { useI18n } from "vue-i18n";
import apiAdmin from "../../../../common/composable/apiAdmin";
import SuperAdminPageHeader from "../../../layouts/SuperAdminPageHeader.vue";
import SubscriptionSidebar from "../SubscriptionSidebar.vue";

export default {
    components: {
        EyeOutlined,
        PlusOutlined,
        EditOutlined,
        DeleteOutlined,
        ExclamationCircleOutlined,
        SaveOutlined,

        SubscriptionSidebar,
        SuperAdminPageHeader,
    },
    setup() {
        const { addEditRequestAdmin, loading, rules } = apiAdmin();
        const { t } = useI18n();
        const formData = ref({});
        const webhookUrl = ref("");

        onMounted(() => {
            axiosAdmin.get("superadmin/payment-settings/stripe").then((response) => {
                formData.value = response.data.data;
                webhookUrl.value = response.data.webhook_url;
            });
        });

        const onSubmit = () => {
            addEditRequestAdmin({
                url: `superadmin/payment-settings/stripe/update`,
                data: formData.value,
                successMessage: t("payment_settings.credential_saved"),
                success: (res) => {},
            });
        };

        return {
            loading,
            rules,
            formData,
            webhookUrl,

            onSubmit,
        };
    },
};
</script>
