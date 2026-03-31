<template>
    <div class="certificate-form">
        <div class="form-card">
            <h2>Fetch Root CA Certificate</h2>

            <form @submit.prevent="handleSubmit">
                <div class="input-group">
                    <label for="url">Domain URL</label>
                    <input id="url" type="url" v-model="url" placeholder="https://example.com" :disabled="loading"
                        required />
                </div>

                <button type="submit" :disabled="loading || !url">
                    <span v-if="loading" class="spinner"></span>
                    {{ loading ? 'Fetching...' : 'Get Root CA' }}
                </button>
            </form>

            <div v-if="error" class="message error">
                <span class="icon">!</span>
                {{ error }}
            </div>

            <div v-if="success" class="message success">
                <span class="icon">✓</span>
                <span>
                    Root CA for <strong>{{ domain }}</strong> retrieved successfully!<br>
                    Chain length: {{ chainLength }} certificate(s)
                </span>
            </div>

            <div v-if="pemContent" class="pem-preview">
                <div class="pem-header">
                    <span>Certificate Preview</span>
                    <button @click="downloadPem" class="download-btn">
                        Download .pem
                    </button>
                </div>
                <pre>{{ pemPreview }}</pre>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const url = ref<string>('')
const loading = ref<boolean>(false)
const error = ref<string>('')
const success = ref<boolean>(false)
const pemContent = ref<string>('')
const domain = ref<string>('')
const chainLength = ref<number>(0)

const pemPreview = computed((): string => {
    if (!pemContent.value) return ''
    const lines = pemContent.value.split('\n')
    return lines.slice(0, 10).join('\n') + '\n...'
})

interface CertificateResponse {
    success: boolean
    domain: string
    pem: string
    chain_length: number
    error?: string
}

const handleSubmit = async (): Promise<void> => {
    if (!url.value) return

    loading.value = true
    error.value = ''
    success.value = false
    pemContent.value = ''
    domain.value = ''
    chainLength.value = 0

    try {
        const response = await fetch('/api/certificate', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ url: url.value })
        })

        const data: CertificateResponse = await response.json()

        if (!response.ok || !data.success) {
            throw new Error(data.error || 'Failed to fetch certificate')
        }

        domain.value = data.domain
        pemContent.value = data.pem
        chainLength.value = data.chain_length
        success.value = true

    } catch (err: unknown) {
        error.value = err instanceof Error ? err.message : 'An unknown error occurred'
    } finally {
        loading.value = false
    }
}

const downloadPem = (): void => {
    if (!pemContent.value || !domain.value) return

    const blob = new Blob([pemContent.value], { type: 'application/x-x509-ca-cert' })
    const downloadUrl = window.URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = downloadUrl
    link.download = `${domain.value}-root-ca.pem`
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    window.URL.revokeObjectURL(downloadUrl)
}
</script>

<style scoped>
.certificate-form {
    width: 100%;
    max-width: 500px;
}

.form-card {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 16px;
    padding: 2rem;
    border: 1px solid rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
}

h2 {
    margin: 0 0 1.5rem 0;
    font-size: 1.5rem;
    font-weight: 600;
    text-align: center;
}

form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.input-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

label {
    font-size: 0.875rem;
    color: #a0a0a0;
    font-weight: 500;
}

input {
    padding: 0.875rem 1rem;
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 8px;
    background: rgba(255, 255, 255, 0.05);
    color: #ffffff;
    font-size: 1rem;
    transition: all 0.2s;
}

input:focus {
    outline: none;
    border-color: #4a9eff;
    background: rgba(255, 255, 255, 0.08);
}

input::placeholder {
    color: #666;
}

input:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

button {
    padding: 0.875rem 1.5rem;
    border: none;
    border-radius: 8px;
    background: linear-gradient(135deg, #4a9eff 0%, #0066ff 100%);
    color: #ffffff;
    font-size: 1rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
}

button:hover:not(:disabled) {
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(74, 158, 255, 0.4);
}

button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    transform: none;
}

.spinner {
    width: 16px;
    height: 16px;
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-top-color: #ffffff;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}

.message {
    padding: 1rem;
    border-radius: 8px;
    display: flex;
    align-items: flex-start;
    gap: 0.75rem;
    font-size: 0.875rem;
    line-height: 1.5;
}

.message.error {
    background: rgba(255, 82, 82, 0.1);
    border: 1px solid rgba(255, 82, 82, 0.3);
    color: #ff6b6b;
}

.message.success {
    background: rgba(76, 209, 55, 0.1);
    border: 1px solid rgba(76, 209, 55, 0.3);
    color: #5cd643;
}

.message .icon {
    font-weight: bold;
    flex-shrink: 0;
}

.message strong {
    color: inherit;
}

.pem-preview {
    margin-top: 1.5rem;
    background: rgba(0, 0, 0, 0.3);
    border-radius: 8px;
    overflow: hidden;
}

.pem-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.75rem 1rem;
    background: rgba(255, 255, 255, 0.05);
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.pem-header span {
    font-size: 0.875rem;
    color: #a0a0a0;
}

.download-btn {
    padding: 0.5rem 1rem;
    font-size: 0.875rem;
    background: rgba(76, 209, 55, 0.2);
    border: 1px solid rgba(76, 209, 55, 0.4);
}

.download-btn:hover {
    background: rgba(76, 209, 55, 0.3);
}

pre {
    padding: 1rem;
    font-family: 'SF Mono', Monaco, 'Cascadia Code', monospace;
    font-size: 0.75rem;
    color: #a0a0a0;
    overflow-x: auto;
    white-space: pre-wrap;
    word-break: break-all;
}
</style>
