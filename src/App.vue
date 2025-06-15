<template>
	<div class="editor-container">
		<div id="editor" ref="editorRef"></div>
	</div>
	<div class="online-users">
		<div class="user-title">在线用户</div>
		<div v-for="user in onlineUsers" :key="user.name" class="user-item">
			<span
				class="user-dot"
				:style="{ backgroundColor: user.color }"
			></span>
			<span class="user-name">{{ user.name }}</span>
		</div>
	</div>
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
	'OC-demo-YL-1231dfsa',
	ydoc
);

const TOOLBAR_CONFIG = [
	[{ header: [] }],
	['bold', 'italic', 'underline', 'link'],
	[{ list: 'ordered' }, { list: 'bullet' }],
	['clean'],
	['file', 'image', 'video'],
];

const onlineUsers = ref<Array<{ name: string; color: string }>>([]);

onMounted(() => {
	let quill: TinyEditor | null = null;
	if (editorRef.value) {
		quill = new TinyEditor(editorRef.value, {
			theme: 'snow',
			modules: {
				cursors: true,
				toolbar: TOOLBAR_CONFIG,
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
			name: 'User_' + Math.random().toString(36).slice(2, 8),
			color,
		});

		// 添加用户状态变化监听
		awareness.on('change', () => {
			const states = Array.from(awareness.getStates().values());
			onlineUsers.value = states.map((state: any) => state.user);
		});

		new QuillBinding(ytext, quill, awareness);

		const persistence = new IndexeddbPersistence('OC-demo-YL-1231dfsa', ydoc);
		persistence.once('synced', (e: any) => {
			console.log('synced with IndexedDB', e);
			console.log('initial content loaded');
		});
	}
});
</script>

<style lang="less" scoped>
.online-users {
	min-width: 200px;
	padding: 16px;
	border-radius: 8px;
	background: #f5f5f5;

	.user-title {
		font-weight: bold;
		margin-bottom: 12px;
	}

	.user-item {
		display: flex;
		align-items: center;
		margin-bottom: 8px;

		.user-dot {
			width: 8px;
			height: 8px;
			border-radius: 50%;
			margin-right: 8px;
		}

		.user-name {
			font-size: 14px;
		}
	}
}
</style>
