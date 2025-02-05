<template>
    <div>
        <label>ФИО:</label>
        <input v-model="form.fullName" type="text" />

        <label>Должность:</label>
        <input v-model="form.position" type="text" />

        <label>Телефон:</label>
        <input v-model="form.phone" type="text" />

        <button @click="previewPDF">Предварительный просмотр PDF</button>
        <button @click="savePDF" :disabled="!pdfDoc">Сохранить PDF</button>

        <div v-if="pdfUrl">
            <pdf :src="pdfUrl"></pdf>
        </div>
    </div>
</template>

<script>
import { ref } from 'vue'
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'
import { PDFViewer } from 'pdfjs-dist/web/pdf_viewer'

export default {
    components: {
        pdf: PDFViewer
    },
    setup() {
        const form = ref({
            fullName: '',
            position: '',
            phone: '',
        })
        const pdfDoc = ref(null)
        const pdfUrl = ref('')

        const previewPDF = async () => {
            const doc = await PDFDocument.create()
            const timesRomanFont = await doc.embedFont(StandardFonts.TimesRoman)

            const page = doc.addPage()
            const {/* width, */height } = page.getSize()
            const fontSize = 30
            page.drawText(`ФИО: ${form.value.fullName}`, {
                x: 50,
                y: height - 4 * fontSize,
                size: fontSize,
                font: timesRomanFont,
                color: rgb(0, 0, 0),
            })
            page.drawText(`Должность: ${form.value.position}`, {
                x: 50,
                y: height - 6 * fontSize,
                size: fontSize,
                font: timesRomanFont,
                color: rgb(0, 0, 0),
            })
            page.drawText(`Телефон: ${form.value.phone}`, {
                x: 50,
                y: height - 8 * fontSize,
                size: fontSize,
                font: timesRomanFont,
                color: rgb(0, 0, 0),
            })

            const pdfBytes = await doc.save()
            const blob = new Blob([pdfBytes], { type: 'application/pdf' })
            pdfUrl.value = URL.createObjectURL(blob)
            pdfDoc.value = pdfBytes
        }

        const savePDF = () => {
            const blob = new Blob([pdfDoc.value], { type: 'application/pdf' })
            const link = document.createElement('a')
            link.href = URL.createObjectURL(blob)
            link.download = 'form-data.pdf'
            link.click()
            link.remove()
        }

        return { form, previewPDF, savePDF, pdfUrl }
    }
}
</script>