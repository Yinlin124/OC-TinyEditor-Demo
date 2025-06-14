<template>
	<div id="editor" ref="editorRef"></div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import Quill from 'quill';
import QuillCursors from 'quill-cursors';
import { WebsocketProvider } from 'y-websocket';
import * as Y from 'yjs';
import { QuillBinding } from 'y-quill';
import { IndexeddbPersistence } from 'y-indexeddb';
import TinyEditor from '@opentiny/fluent-editor';

Quill.register('modules/cursors', QuillCursors);

const editorRef = ref<HTMLDivElement | null>(null);

const ydoc = new Y.Doc();
const provider = new WebsocketProvider(
	'wss://demos.yjs.dev/ws',
	'OC-demo-YL',
	ydoc
);
onMounted(() => {
	let quill: TinyEditor | null = null;
	if (editorRef.value) {
		quill = new TinyEditor(editorRef.value, {
			theme: 'snow',
			modules: {
				cursors: true,
			},
		});
	}
	if (quill) {
		const ytext = ydoc.getText('quill');
		const awareness = provider.awareness;

		// 设置本地用户信息
		let color =
			'#' + Math.random().toString(16).split('.')[1].slice(0, 6);
		awareness.setLocalStateField('user', {
			name: 'User_' + Math.random().toString(36).substr(2, 8),
			color,
		});

		new QuillBinding(ytext, quill, awareness);

		awareness.on('change', (changes: Yjs.Transaction) => {
			// 获取所有协同信息 用户列表
			const allUsers = Array.from(awareness.getStates().values()).map(
				item => item.user
			);
			console.log('changes', changes, allUsers);
		});

		const persistence = new IndexeddbPersistence('OC-demo-YL', ydoc);
		persistence.once('synced', (e: any) => {
			console.log('synced with IndexedDB', e);
			console.log('initial content loaded');
		});
	}
});
</script>

<style lang="less" scoped></style>
