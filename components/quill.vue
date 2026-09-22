<template>
    <div :id="quill_id" class="quill"></div>
</template>

<script setup>
    let props = defineProps(['modelValue'])
    let emit = defineEmits(['update:modelValue', 'input'])
    
    let quill = null,
        quill_dom = null,
        quill_id = ref('quill_' + random())
    
    // 图片直传 OSS，编辑器内插入解析后的URL（提交时服务端归一为 static://）
    const quillImageHandler = () => {
        let input = document.createElement('input')
        input.type = 'file'
        input.accept = 'image/jpeg,image/png,image/gif,image/webp'
        input.onchange = async () => {
            let file = input.files[0]
            if (! file) return
            let lmsgID = loadingMsg('<strong>图片上传中...</strong>', 0)
            try {
                let r = await ossUpload(file, 'forum', {}, (pct) => editLoadingMsg_percent(lmsgID, pct))
                let range = quill.getSelection(true)
                quill.insertEmbed(range.index, 'image', r.url)
                quill.setSelection(range.index + 1)
            } catch (ex) {
                errMsg('<strong>图片上传失败：</strong><br/><span>' + ex + '</span>')
            }
            closeLoadingMsg(lmsgID)
        }
        input.click()
    }
    
    onMounted(() => {
        /* quill配置参考：https://www.jianshu.com/p/b237372f15cc */
        quill = new Quill('#' + quill_id.value, {
            modules: {
                toolbar: {
                    container: [
                        [{ 'size': ['small', 'large', 'huge', false] }],
                        
                        ['bold', 'italic', 'underline', 'strike'],
                        [{ 'list': 'ordered'}, { 'list': 'bullet' }],
                        [{ 'script': 'sub'}, { 'script': 'super' }],
                        [{ 'indent': '-1'}, { 'indent': '+1' }],
                        
                        [{ 'color': [] }, { 'background': [] }],
                        [{ 'align': [] }],
                        ['link', 'image'],
                        
                        ['clean']
                    ],
                    handlers: {
                        image: quillImageHandler
                    }
                }
            },
            theme: 'snow'
        })
        quill_dom = document.getElementById(quill_id.value).firstChild
        quill.on('text-change', function() {
            let ncontent = quill_dom.innerHTML
            emit('update:modelValue', ncontent)
            emit('input')
        })
    })
    watch(() => props.modelValue, () => {
        if (! import.meta.client) return
        
        // 防止不是用户输入改变quill本身内容触发（自建的v-model导致）的，否则会导致用户输入不了
        let ncontent = quill_dom.innerHTML
        if (props.modelValue == ncontent) return
        quill_dom.innerHTML = props.modelValue
    })
</script>