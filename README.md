# CHATPDF CODE

import React, { useState, useRef, useEffect } from 'react';
import { View, Text, StyleSheet, TextInput, TouchableOpacity, ScrollView, Modal, Platform, KeyboardAvoidingView } from 'react-native';
import { LinearGradient } from 'expo-linear-gradient';
import { Ionicons } from '@expo/vector-icons';

type Message = {
  id: number;
  text: string;
  sender: 'user' | 'bot';
};

export default function HomeScreen() {
  const [pdfUploaded, setPdfUploaded] = useState<boolean>(false);
  const [pdfUrl, setPdfUrl] = useState<string>('');
  const [pdfTitle, setPdfTitle] = useState<string>('');
  const [showUploadModal, setShowUploadModal] = useState<boolean>(false);
  const [uploadInput, setUploadInput] = useState<string>('');
  const [messages, setMessages] = useState<Message[]>([]);
  const [inputMessage, setInputMessage] = useState<string>('');
  const scrollViewRef = useRef<ScrollView>(null);

  useEffect(() => {
    if (scrollViewRef.current) {
      scrollViewRef.current.scrollToEnd({ animated: true });
    }
  }, [messages]);

  const handlePdfUpload = () => {
    if (uploadInput.trim() === '') return;
    setPdfUrl(uploadInput);
    // Extract title from URL (displaying the last segment as title)
    const parts = uploadInput.split('/');
    const title = parts[parts.length - 1] || 'Uploaded PDF';
    setPdfTitle(title);
    setUploadInput('');
    setShowUploadModal(false);
    setPdfUploaded(true);
    // Welcome message from the bot after successful upload.
    setMessages([
      { id: Date.now(), text: `PDF "${title}" uploaded successfully! Ask me anything about its content.`, sender: 'bot' },
    ]);
  };

  const handleSendMessage = () => {
    if (inputMessage.trim() === '') return;
    const userMessage: Message = {
      id: Date.now(),
      text: inputMessage,
      sender: 'user',
    };
    setMessages(prev => [...prev, userMessage]);
    setInputMessage('');
    // Simulate a bot response after a brief delay.
    setTimeout(() => {
      const botResponse: Message = {
        id: Date.now() + 1,
        text: simulateBotResponse(userMessage.text),
        sender: 'bot',
      };
      setMessages(prev => [...prev, botResponse]);
    }, 1500);
  };

  const simulateBotResponse = (userText: string): string => {
    // This is a simulated reply. In a production app, you'd query an AI service.
    return `I processed your question: "${userText}". (This is a dummy response based on the content of "${pdfTitle}")`;
  };

  const renderUploadScreen = () => {
    return (
      <View style={styles.uploadContainer}>
        <Ionicons name="document-text-outline" size={80} color="#3b5998" />
        <Text style={styles.uploadText}>Upload a PDF to start chatting</Text>
        <TouchableOpacity style={styles.uploadButton} onPress={() => setShowUploadModal(true)}>
          <Text style={styles.uploadButtonText}>Upload PDF</Text>
        </TouchableOpacity>
      </View>
    );
  };

  const renderChatScreen = () => {
    return (
      <View style={styles.chatContainer}>
        <View style={styles.chatHeader}>
          <Text style={styles.chatTitle}>{pdfTitle}</Text>
          <TouchableOpacity style={styles.changePdfButton} onPress={() => setPdfUploaded(false)}>
            <Ionicons name="refresh-circle" size={24} color="#fff" />
            <Text style={styles.changePdfText}>Change PDF</Text>
          </TouchableOpacity>
        </View>
        <ScrollView style={styles.messagesContainer} ref={scrollViewRef} contentContainerStyle={{ paddingVertical: 10 }}>
          {messages.map(message => (
            <View
              key={message.id}
              style={[
                styles.messageBubble,
                message.sender === 'user' ? styles.userBubble : styles.botBubble,
              ]}
            >
              <Text style={message.sender === 'user' ? styles.userText : styles.botText}>
                {message.text}
              </Text>
            </View>
          ))}
        </ScrollView>
        <View style={styles.inputArea}>
          <TextInput
            style={styles.textInput}
            value={inputMessage}
            onChangeText={setInputMessage}
            placeholder="Type your message..."
            placeholderTextColor="#888"
          />
          <TouchableOpacity style={styles.sendButton} onPress={handleSendMessage}>
            <Ionicons name="send" size={24} color="#fff" />
          </TouchableOpacity>
        </View>
      </View>
    );
  };

  return (
    <LinearGradient colors={['#4c669f', '#3b5998', '#192f6a']} style={styles.container}>
      <KeyboardAvoidingView behavior={Platform.OS === 'ios' ? 'padding' : undefined} style={styles.flexContainer}>
        <View style={styles.header}>
          <Text style={styles.headerTitle}>ChatPDF Clone</Text>
        </View>
        <View style={styles.content}>
          {pdfUploaded ? renderChatScreen() : renderUploadScreen()}
        </View>
      </KeyboardAvoidingView>
      <Modal visible={showUploadModal} transparent animationType="slide">
        <View style={styles.modalBackground}>
          <View style={styles.modalContainer}>
            <Text style={styles.modalTitle}>Enter PDF URL</Text>
            <TextInput
              style={styles.modalInput}
              value={uploadInput}
              onChangeText={setUploadInput}
              placeholder="https://example.com/file.pdf"
              placeholderTextColor="#888"
            />
            <View style={styles.modalButtons}>
              <TouchableOpacity style={styles.modalButton} onPress={handlePdfUpload}>
                <Text style={styles.modalButtonText}>Upload</Text>
              </TouchableOpacity>
              <TouchableOpacity style={styles.modalCancel} onPress={() => setShowUploadModal(false)}>
                <Text style={styles.modalCancelText}>Cancel</Text>
              </TouchableOpacity>
            </View>
          </View>
        </View>
      </Modal>
    </LinearGradient>
  );
}

const styles = StyleSheet.create({
  flexContainer: {
    flex: 1,
  },
  container: {
    flex: 1,
  },
  header: {
    paddingTop: 40,
    paddingBottom: 20,
    alignItems: 'center',
    backgroundColor: 'transparent',
  },
  headerTitle: {
    color: '#fff',
    fontSize: 24,
    fontWeight: 'bold',
  },
  content: {
    flex: 1,
    paddingHorizontal: 16,
    paddingBottom: 10,
  },
  uploadContainer: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  uploadText: {
    fontSize: 18,
    color: '#fff',
    marginVertical: 20,
    textAlign: 'center',
  },
  uploadButton: {
    backgroundColor: '#fff',
    paddingHorizontal: 20,
    paddingVertical: 10,
    borderRadius: 25,
  },
  uploadButtonText: {
    color: '#3b5998',
    fontSize: 16,
    fontWeight: 'bold',
  },
  chatContainer: {
    flex: 1,
    backgroundColor: '#f2f2f2',
    borderRadius: 8,
    overflow: 'hidden',
  },
  chatHeader: {
    backgroundColor: '#3b5998',
    padding: 16,
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  chatTitle: {
    color: '#fff',
    fontSize: 18,
    fontWeight: 'bold',
  },
  changePdfButton: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  changePdfText: {
    color: '#fff',
    marginLeft: 4,
    fontSize: 14,
  },
  messagesContainer: {
    flex: 1,
    paddingHorizontal: 10,
  },
  messageBubble: {
    maxWidth: '80%',
    marginVertical: 6,
    padding: 10,
    borderRadius: 10,
  },
  userBubble: {
    backgroundColor: '#3b5998',
    alignSelf: 'flex-end',
    borderTopRightRadius: 0,
  },
  botBubble: {
    backgroundColor: '#fff',
    alignSelf: 'flex-start',
    borderTopLeftRadius: 0,
    borderWidth: 1,
    borderColor: '#ddd',
  },
  userText: {
    color: '#fff',
  },
  botText: {
    color: '#333',
  },
  inputArea: {
    flexDirection: 'row',
    padding: 10,
    backgroundColor: '#eee',
    alignItems: 'center',
  },
  textInput: {
    flex: 1,
    backgroundColor: '#fff',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 20,
    marginRight: 10,
    borderWidth: 1,
    borderColor: '#ddd',
  },
  sendButton: {
    backgroundColor: '#3b5998',
    padding: 12,
    borderRadius: 25,
  },
  modalBackground: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.5)',
    justifyContent: 'center',
    padding: 20,
  },
  modalContainer: {
    backgroundColor: '#fff',
    borderRadius: 8,
    padding: 20,
  },
  modalTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
    textAlign: 'center',
  },
  modalInput: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    paddingHorizontal: 12,
    paddingVertical: 8,
    marginBottom: 20,
    color: '#333',
  },
  modalButtons: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  modalButton: {
    backgroundColor: '#3b5998',
    paddingHorizontal: 20,
    paddingVertical: 10,
    borderRadius: 25,
  },
  modalButtonText: {
    color: '#fff',
    fontWeight: 'bold',
  },
  modalCancel: {
    paddingHorizontal: 20,
    paddingVertical: 10,
    borderRadius: 25,
    borderWidth: 1,
    borderColor: '#3b5998',
  },
  modalCancelText: {
    color: '#3b5998',
    fontWeight: 'bold',
  },
});
