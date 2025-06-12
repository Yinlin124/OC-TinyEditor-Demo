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

onMounted(() => {
	let quill: TinyEditor | null = null;
	if (editorRef.value) {
		quill = new TinyEditor(editorRef.value, {
			theme: 'snow',
		});
	}
	if (quill) {
		const ydoc = new Y.Doc();
		// connect to the public demo server (not in production!)
		const provider = new WebsocketProvider(
			'wss://demos.yjs.dev/ws',
			'quill-demo-room',
			ydoc
		);
		// Define a shared text type on the document
		const ytext = ydoc.getText('quill');
		const binding = new QuillBinding(ytext, quill, provider.awareness);
		const persistence = new IndexeddbPersistence('quill-demo-room', ydoc);
		persistence.once('synced', (e: any) => {
			console.log('synced with IndexedDB', e);
			// Load initial content from IndexedDB
			console.log('initial content loaded');
		});
	}
});
</script>

<style lang="scss" scoped></style>
