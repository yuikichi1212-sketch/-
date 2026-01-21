import React, { useState, useEffect, useRef } from 'react';
import { base44 } from '@/api/base44Client';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { AnimatePresence } from 'framer-motion';
import { Palette, Plus, Menu, Phone, User as UserIcon, Search, Pin, Video, Bell, Smile, Home, LogOut, UserPlus } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter, DialogDescription } from '@/components/ui/dialog';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
} from "@/components/ui/alert-dialog";
import { toast } from 'sonner';

import MessageBubble from '../components/chat/MessageBubble';
import ChatInput from '../components/chat/ChatInput';
import ThemeCustomizer from '../components/chat/ThemeCustomizer';
import AIAssistant from '../components/chat/AIAssistant';
import RoomList from '../components/chat/RoomList';
import ProfileSettings from '../components/profile/ProfileSettings';
import UserProfileView from '../components/chat/UserProfileView';
import PasswordDialog from '../components/chat/PasswordDialog';
import SearchMessages from '../components/chat/SearchMessages';
import MessageActions from '../components/chat/MessageActions';
import TypingIndicator from '../components/chat/TypingIndicator';
import QuickActions from '../components/chat/QuickActions';
import VideoCallDialog from '../components/chat/VideoCallDialog';
import StampPicker from '../components/chat/StampPicker';
import NotificationBadge from '../components/chat/NotificationBadge';
import NotificationSettings from '../components/settings/NotificationSettings';

const THEME_PRESETS = {
  glass: { primary: '#64748b', secondary: '#475569', gradient: 'linear-gradient(135deg, #f1f5f9, #e2e8f0)' },
  pastel: { primary: '#FF6B9D', secondary: '#C44569', gradient: 'linear-gradient(135deg, #FFD6E8, #C4DDFF)' },
  dark: { primary: '#667EEA', secondary: '#764BA2', gradient: 'linear-gradient(135deg, #2D3748, #1A202C)' },
  neon: { primary: '#00F5FF', secondary: '#FF00E5', gradient: 'linear-gradient(135deg, #0F0F0F, #1A1A2E)' },
  ocean: { primary: '#00D4FF', secondary: '#0096C7', gradient: 'linear-gradient(135deg, #E0F7FF, #B8E6F5)' },
  sunset: { primary: '#FF6B35', secondary: '#F7931E', gradient: 'linear-gradient(135deg, #FFE5D9, #FFDAB9)' },
  forest: { primary: '#52B788', secondary: '#2D6A4F', gradient: 'linear-gradient(135deg, #D8F3DC, #B7E4C7)' }
};

export default function ChatPage() {
  const queryClient = useQueryClient();
  const messagesEndRef = useRef(null);
  const [user, setUser] = useState(null);
  const [activeRoomId, setActiveRoomId] = useState(null);
  const [pendingRoomId, setPendingRoomId] = useState(null);
  const [showThemeCustomizer, setShowThemeCustomizer] = useState(false);
  const [showRoomDialog, setShowRoomDialog] = useState(false);
  const [showRoomList, setShowRoomList] = useState(false);
  const [showProfileSettings, setShowProfileSettings] = useState(false);
  const [showSearch, setShowSearch] = useState(false);
  const [showVideoCall, setShowVideoCall] = useState(false);
  const [showStampPicker, setShowStampPicker] = useState(false);
  const [showNotificationSettings, setShowNotificationSettings] = useState(false);
  const [viewingUserEmail, setViewingUserEmail] = useState(null);
  const [unreadCount, setUnreadCount] = useState(0);
  const [showInviteDialog, setShowInviteDialog] = useState(false);
  const [inviteEmail, setInviteEmail] = useState('');
  const [editingRoom, setEditingRoom] = useState(null);
  const [editingMessage, setEditingMessage] = useState(null);
  const [roomToDelete, setRoomToDelete] = useState(null);
  const [messageToDelete, setMessageToDelete] = useState(null);
  const [roomFormData, setRoomFormData] = useState({ name: '', phone_number: '', password: '' });
  const [replyTo, setReplyTo] = useState(null);
  const [blockedUsers, setBlockedUsers] = useState([]);
  const [accessedRooms, setAccessedRooms] = useState([]);
  const [isTyping, setIsTyping] = useState(false);
  const [typingTimeout, setTypingTimeout] = useState(null);

  const { data: userTheme } = useQuery({
    queryKey: ['userTheme'],
    queryFn: async () => {
      const themes = await base44.entities.UserTheme.filter({ created_by: (await base44.auth.me()).email });
      return themes[0] || {
        theme_name: 'glass',
        primary_color: '#64748b',
        secondary_color: '#475569',
        background_style: 'gradient',
        bubble_style: 'rounded',
        font_size: 'medium'
      };
    },
    initialData: {
      theme_name: 'glass',
      primary_color: '#64748b',
      secondary_color: '#475569',
      background_style: 'gradient',
      bubble_style: 'rounded',
      font_size: 'medium'
    }
  });

  const { data: rooms = [] } = useQuery({
    queryKey: ['chatRooms'],
    queryFn: () => base44.entities.ChatRoom.list('-updated_date'),
    initialData: []
  });

  const { data: allMessages = [] } = useQuery({
    queryKey: ['messages', activeRoomId],
    queryFn: () => activeRoomId ? base44.entities.Message.filter({ chat_room_id: activeRoomId }, 'created_date') : [],
    enabled: !!activeRoomId,
    initialData: [],
    refetchInterval: 3000
  });

  useEffect(() => {
    const loadBlockedUsers = async () => {
      if (!user) return;
      try {
        const blocks = await base44.entities.BlockedUser.filter({ blocker_email: user.email });
        setBlockedUsers(blocks.map(b => b.blocked_email));
      } catch (error) {
        console.error('ブロックリスト読み込みエラー:', error);
      }
    };
    loadBlockedUsers();
  }, [user]);

  const messages = allMessages.filter(m => !blockedUsers.includes(m.sender_email));

  useEffect(() => {
    if (!user) return;
    const unread = messages.filter(m => 
      m.sender_email !== user.email && 
      !m.read_by?.includes(user.email)
    ).length;
    setUnreadCount(unread);
    
    if (unread > 0) {
      document.title = `(${unread}) ゆいきちチャット`;
    } else {
      document.title = 'ゆいきちチャット';
    }
  }, [messages, user]);

  useEffect(() => {
    const loadUser = async () => {
      try {
        const currentUser = await base44.auth.me();
        setUser(currentUser);
      } catch (error) {
        console.error('ユーザー取得エラー:', error);
      }
    };
    loadUser();
  }, []);

  useEffect(() => {
    if (rooms.length > 0 && !activeRoomId) {
      const firstRoom = rooms[0];
      if (firstRoom.password && !accessedRooms.includes(firstRoom.id)) {
        setPendingRoomId(firstRoom.id);
      } else {
        setActiveRoomId(firstRoom.id);
      }
    }
  }, [rooms, activeRoomId, accessedRooms]);

  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  useEffect(() => {
    const markAsRead = async () => {
      if (!activeRoomId || !user) return;
      const unreadMessages = messages.filter(m => 
        m.sender_email !== user.email && 
        !m.read_by?.includes(user.email)
      );
      for (const msg of unreadMessages) {
        await base44.entities.Message.update(msg.id, {
          read_by: [...(msg.read_by || []), user.email]
        });
      }
    };
    markAsRead();
  }, [messages, activeRoomId, user]);

  const updateThemeMutation = useMutation({
    mutationFn: async (themeUpdate) => {
      if (userTheme.id) {
        return await base44.entities.UserTheme.update(userTheme.id, themeUpdate);
      } else {
        return await base44.entities.UserTheme.create(themeUpdate);
      }
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['userTheme'] });
    }
  });

  const saveRoomMutation = useMutation({
    mutationFn: async (roomData) => {
      if (editingRoom) {
        return await base44.entities.ChatRoom.update(editingRoom.id, roomData);
      } else {
        return await base44.entities.ChatRoom.create({
          ...roomData,
          participants: [user.email]
        });
      }
    },
    onSuccess: (savedRoom) => {
      queryClient.invalidateQueries({ queryKey: ['chatRooms'] });
      if (!editingRoom) {
        setActiveRoomId(savedRoom.id);
      }
      setShowRoomDialog(false);
      setShowRoomList(false);
      setRoomFormData({ name: '', phone_number: '' });
      setEditingRoom(null);
      toast.success(editingRoom ? 'ルームを更新しました！' : 'ルームを作成しました！');
    }
  });

  const deleteRoomMutation = useMutation({
    mutationFn: async (roomId) => {
      const messagesToDelete = await base44.entities.Message.filter({ chat_room_id: roomId });
      await Promise.all(messagesToDelete.map(m => base44.entities.Message.delete(m.id)));
      return await base44.entities.ChatRoom.delete(roomId);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['chatRooms'] });
      if (roomToDelete?.id === activeRoomId) {
        setActiveRoomId(null);
      }
      setRoomToDelete(null);
      toast.success('ルームを削除しました');
    }
  });

  const sendMessageMutation = useMutation({
    mutationFn: async (messageData) => {
      const message = await base44.entities.Message.create({
        ...messageData,
        chat_room_id: activeRoomId,
        sender_email: user.email,
        sender_name: user.full_name,
        reactions: []
      });
      
      await base44.entities.ChatRoom.update(activeRoomId, {
        last_message: messageData.content,
        last_message_time: new Date().toISOString()
      });
      
      return message;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['messages', activeRoomId] });
      queryClient.invalidateQueries({ queryKey: ['chatRooms'] });
      setReplyTo(null);
    }
  });

  const reactToMessageMutation = useMutation({
    mutationFn: async ({ messageId, emoji }) => {
      const message = messages.find(m => m.id === messageId);
      const existingReactions = message.reactions || [];
      const userReactionIndex = existingReactions.findIndex(r => r.user_email === user.email && r.emoji === emoji);
      
      let newReactions;
      if (userReactionIndex >= 0) {
        newReactions = existingReactions.filter((_, i) => i !== userReactionIndex);
      } else {
        newReactions = [...existingReactions, { emoji, user_email: user.email }];
      }
      
      return await base44.entities.Message.update(messageId, { reactions: newReactions });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['messages', activeRoomId] });
    }
  });

  const updateMessageMutation = useMutation({
    mutationFn: async ({ messageId, content }) => {
      return await base44.entities.Message.update(messageId, { content, edited: true });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['messages', activeRoomId] });
      setEditingMessage(null);
      toast.success('メッセージを編集しました');
    }
  });

  const deleteMessageMutation = useMutation({
    mutationFn: async (messageId) => {
      return await base44.entities.Message.delete(messageId);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['messages', activeRoomId] });
      setMessageToDelete(null);
      toast.success('メッセージを削除しました');
    }
  });

  const pinMessageMutation = useMutation({
    mutationFn: async (messageId) => {
      return await base44.entities.ChatRoom.update(activeRoomId, { pinned_message_id: messageId });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['chatRooms'] });
      toast.success('メッセージをピン留めしました');
    }
  });

  const themeColors = {
    primary: userTheme.primary_color,
    secondary: userTheme.secondary_color
  };

  const backgroundStyle = userTheme.background_style === 'image' && userTheme.background_image_url
    ? `url(${userTheme.background_image_url}) center/cover`
    : THEME_PRESETS[userTheme.theme_name]?.gradient || THEME_PRESETS.pastel.gradient;

  const handleSuggestionSelect = (suggestion) => {
    sendMessageMutation.mutate({ content: suggestion });
  };

  const handleNewRoom = () => {
    setEditingRoom(null);
    setRoomFormData({ name: '', phone_number: '' });
    setShowRoomDialog(true);
  };

  const handleEditRoom = (room) => {
    setEditingRoom(room);
    setRoomFormData({ 
      name: room.name, 
      phone_number: room.phone_number || '', 
      password: room.password || '' 
    });
    setShowRoomDialog(true);
    setShowRoomList(false);
  };

  const handleRoomSelect = (roomId) => {
    const room = rooms.find(r => r.id === roomId);
    if (room?.password && !accessedRooms.includes(roomId)) {
      setPendingRoomId(roomId);
    } else {
      setActiveRoomId(roomId);
    }
  };

  const handlePasswordSubmit = (password) => {
    const room = rooms.find(r => r.id === pendingRoomId);
    if (room && room.password === password) {
      const newAccessedRooms = [...accessedRooms, pendingRoomId];
      setAccessedRooms(newAccessedRooms);
      localStorage.setItem('accessedRooms', JSON.stringify(newAccessedRooms));
      setActiveRoomId(pendingRoomId);
      setPendingRoomId(null);
      toast.success('入室しました！');
    } else {
      toast.error('パスワードが間違っています');
    }
  };

  useEffect(() => {
    const saved = localStorage.getItem('accessedRooms');
    if (saved) {
      setAccessedRooms(JSON.parse(saved));
    }
  }, []);

  const handleTyping = () => {
    setIsTyping(true);
    if (typingTimeout) clearTimeout(typingTimeout);
    const timeout = setTimeout(() => setIsTyping(false), 2000);
    setTypingTimeout(timeout);
  };

  const handleDeleteRoom = (room) => {
    setRoomToDelete(room);
  };

  const handleCall = () => {
    const currentRoom = rooms.find(r => r.id === activeRoomId);
    if (currentRoom?.phone_number) {
      window.open(`tel:${currentRoom.phone_number}`);
    } else {
      toast.error('電話番号が設定されていません');
    }
  };

  const handleLeaveRoom = async () => {
    if (!currentRoom || !user) return;
    try {
      const updatedParticipants = currentRoom.participants.filter(p => p !== user.email);
      await base44.entities.ChatRoom.update(currentRoom.id, {
        participants: updatedParticipants
      });
      queryClient.invalidateQueries({ queryKey: ['chatRooms'] });
      setActiveRoomId(null);
      toast.success('ルームから退出しました');
    } catch (error) {
      toast.error('退出に失敗しました');
    }
  };

  const handleInviteUser = async () => {
    if (!currentRoom || !inviteEmail.trim()) return;
    try {
      const updatedParticipants = [...(currentRoom.participants || []), inviteEmail];
      await base44.entities.ChatRoom.update(currentRoom.id, {
        participants: updatedParticipants
      });
      queryClient.invalidateQueries({ queryKey: ['chatRooms'] });
      setShowInviteDialog(false);
      setInviteEmail('');
      toast.success('ユーザーを招待しました！');
    } catch (error) {
      toast.error('招待に失敗しました');
    }
  };

  if (!user) {
    return (
      <div className="h-screen flex items-center justify-center">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-gray-900"></div>
      </div>
    );
  }

  const currentRoom = rooms.find(r => r.id === activeRoomId);

  return (
    <div className="h-screen flex overflow-hidden relative" style={{ background: backgroundStyle }}>
      <div className="absolute inset-0 bg-gradient-to-br from-white/30 to-transparent pointer-events-none" />

      <AnimatePresence>
        {showRoomList && (
          <>
            <motion.div
              initial={{ opacity: 0 }}
              animate={{ opacity: 1 }}
              exit={{ opacity: 0 }}
              className="fixed inset-0 bg-black/40 backdrop-blur-sm z-40"
              onClick={() => setShowRoomList(false)}
            />
            <motion.div
              initial={{ opacity: 0, x: -300 }}
              animate={{ opacity: 1, x: 0 }}
              exit={{ opacity: 0, x: -300 }}
              transition={{ type: "spring", damping: 25 }}
              className="fixed left-0 top-0 bottom-0 z-50 w-80 bg-white/95 backdrop-blur-2xl shadow-2xl border-r border-white/20"
            >
              <RoomList
                rooms={rooms}
                activeRoomId={activeRoomId}
                onRoomSelect={(id) => {
                  handleRoomSelect(id);
                  setShowRoomList(false);
                }}
                onNewRoom={handleNewRoom}
                onEditRoom={handleEditRoom}
                onDeleteRoom={handleDeleteRoom}
                themeColors={themeColors}
              />
            </motion.div>
          </>
        )}
      </AnimatePresence>

      <div className="flex-1 flex flex-col relative z-10">
        {activeRoomId ? (
          <>
            <div 
              className="border-b bg-white/70 backdrop-blur-xl shadow-lg border-white/20"
            >
              <div className="p-3 md:p-4 flex justify-between items-center">
              <div className="flex items-center gap-2 md:gap-3 flex-1 min-w-0">
                <Button
                  variant="ghost"
                  size="icon"
                  onClick={() => window.location.href = '/'}
                  className="rounded-full flex-shrink-0"
                >
                  <Home className="w-5 h-5" />
                </Button>
                <Button
                  variant="ghost"
                  size="icon"
                  onClick={() => setShowRoomList(true)}
                  className="md:hidden rounded-full flex-shrink-0"
                >
                  <Menu className="w-5 h-5" />
                </Button>
                <div className="min-w-0 flex-1">
                  <h1 className="text-base md:text-xl font-bold truncate">
                    {currentRoom?.name || 'ゆいきちチャット'}
                  </h1>
                  <p className="text-xs text-gray-500 truncate">
                    {currentRoom?.participants?.length || 0} 人
                  </p>
                </div>
              </div>
              <div className="flex items-center gap-0.5 md:gap-1 flex-shrink-0">
                <Button
                  variant="ghost"
                  size="icon"
                  onClick={() => setShowSearch(true)}
                  className="rounded-full h-8 w-8 md:h-10 md:w-10"
                >
                  <Search className="w-4 h-4 md:w-5 md:h-5" />
                </Button>
                <div className="relative">
                  <Button
                    variant="ghost"
                    size="icon"
                    onClick={() => setShowNotificationSettings(true)}
                    className="rounded-full h-8 w-8 md:h-10 md:w-10"
                  >
                    <Bell className="w-4 h-4 md:w-5 md:h-5" />
                  </Button>
                  <NotificationBadge count={unreadCount} />
                </div>
                <Button
                  variant="ghost"
                  size="icon"
                  onClick={() => setShowVideoCall(true)}
                  className="rounded-full h-8 w-8 md:h-10 md:w-10"
                  style={{ color: themeColors.primary }}
                >
                  <Video className="w-4 h-4 md:w-5 md:h-5" />
                </Button>
                {currentRoom?.phone_number && (
                  <Button
                    variant="ghost"
                    size="icon"
                    onClick={handleCall}
                    className="rounded-full h-8 w-8 md:h-10 md:w-10"
                    style={{ color: themeColors.secondary }}
                  >
                    <Phone className="w-4 h-4 md:w-5 md:h-5" />
                  </Button>
                )}
                <Button
                  variant="ghost"
                  size="icon"
                  onClick={() => setShowProfileSettings(true)}
                  className="rounded-full h-8 w-8 md:h-10 md:w-10"
                >
                  <UserIcon className="w-4 h-4 md:w-5 md:h-5" />
                </Button>
                <Button
                  variant="ghost"
                  size="icon"
                  onClick={() => setShowThemeCustomizer(true)}
                  className="rounded-full h-8 w-8 md:h-10 md:w-10 flex-shrink-0"
                >
                  <Palette className="w-4 h-4 md:w-5 md:h-5" />
                </Button>
                <Button
                  variant="ghost"
                  size="icon"
                  onClick={() => setShowInviteDialog(true)}
                  className="rounded-full h-8 w-8 md:h-10 md:w-10"
                  style={{ color: themeColors.primary }}
                >
                  <UserPlus className="w-4 h-4 md:w-5 md:h-5" />
                </Button>
                <Button
                  variant="ghost"
                  size="icon"
                  onClick={handleLeaveRoom}
                  className="rounded-full h-8 w-8 md:h-10 md:w-10 text-red-500 hover:text-red-600"
                >
                  <LogOut className="w-4 h-4 md:w-5 md:h-5" />
                </Button>
              </div>
              </div>
              
              {currentRoom && <QuickActions room={currentRoom} themeColors={themeColors} />}
            </div>

            <div className="flex-1 overflow-y-auto p-3 md:p-4 space-y-2 pb-4">
              {currentRoom?.pinned_message_id && (
                <div className="sticky top-0 z-10 mb-4">
                  <div className="bg-yellow-50 border border-yellow-200 rounded-lg p-3 flex items-start gap-2">
                    <Pin className="w-4 h-4 text-yellow-600 mt-0.5" />
                    <div className="flex-1">
                      <p className="text-xs font-medium text-yellow-800 mb-1">ピン留めメッセージ</p>
                      <p className="text-sm text-gray-700">
                        {messages.find(m => m.id === currentRoom.pinned_message_id)?.content}
                      </p>
                    </div>
                  </div>
                </div>
              )}
              
              <AnimatePresence>
                {messages.map((message) => (
                  <div key={message.id} id={`message-${message.id}`}>
                    <div onClick={() => {
                      if (message.sender_email !== user.email) {
                        setViewingUserEmail(message.sender_email);
                      }
                    }}>
                      <MessageBubble
                        message={message}
                        isOwn={message.sender_email === user.email}
                        onReact={(messageId, emoji) => reactToMessageMutation.mutate({ messageId, emoji })}
                        onReply={setReplyTo}
                        onEdit={setEditingMessage}
                        onDelete={setMessageToDelete}
                        themeColors={themeColors}
                      />
                    </div>

                  </div>
                ))}
                {isTyping && <TypingIndicator themeColors={themeColors} />}
              </AnimatePresence>
              <div ref={messagesEndRef} />
            </div>

            <ChatInput
              onSend={(data) => {
                if (editingMessage) {
                  updateMessageMutation.mutate({ 
                    messageId: editingMessage.id, 
                    content: data.content 
                  });
                } else {
                  sendMessageMutation.mutate(data);
                }
              }}
              replyTo={replyTo}
              editingMessage={editingMessage}
              onCancelReply={() => setReplyTo(null)}
              onCancelEdit={() => setEditingMessage(null)}
              onTyping={handleTyping}
              onStampClick={() => setShowStampPicker(true)}
              themeColors={themeColors}
            />

            <AIAssistant
              chatRoomId={activeRoomId}
              onSuggestionSelect={handleSuggestionSelect}
              themeColors={themeColors}
            />
          </>
        ) : (
          <div className="flex-1 flex items-center justify-center text-gray-400 p-4">
            <div className="text-center">
              <Plus className="w-12 md:w-16 h-12 md:h-16 mx-auto mb-4 opacity-20" />
              <p className="text-sm md:text-base">チャットルームを選択または作成してください</p>
              <Button
                onClick={() => setShowRoomList(true)}
                className="mt-4 md:hidden"
                style={{
                  background: `linear-gradient(135deg, ${themeColors.primary}, ${themeColors.secondary})`
                }}
              >
                ルームを選択
              </Button>
            </div>
          </div>
        )}
      </div>

      <AnimatePresence>
        {showThemeCustomizer && (
          <ThemeCustomizer
            currentTheme={userTheme}
            onThemeChange={(updates) => updateThemeMutation.mutate(updates)}
            onClose={() => setShowThemeCustomizer(false)}
          />
        )}
        
        {showProfileSettings && (
          <ProfileSettings
            user={user}
            onClose={() => setShowProfileSettings(false)}
            onUpdate={async () => {
              const updatedUser = await base44.auth.me();
              setUser(updatedUser);
            }}
            themeColors={themeColors}
          />
        )}
        
        {viewingUserEmail && (
          <UserProfileView
            userEmail={viewingUserEmail}
            currentUser={user}
            onClose={() => setViewingUserEmail(null)}
            themeColors={themeColors}
          />
        )}

        {pendingRoomId && (
          <PasswordDialog
            room={rooms.find(r => r.id === pendingRoomId)}
            onSubmit={handlePasswordSubmit}
            onCancel={() => setPendingRoomId(null)}
            themeColors={themeColors}
          />
        )}

        {showSearch && (
          <SearchMessages
            messages={allMessages}
            onMessageSelect={(msg) => {
              const element = document.getElementById(`message-${msg.id}`);
              element?.scrollIntoView({ behavior: 'smooth', block: 'center' });
            }}
            onClose={() => setShowSearch(false)}
            themeColors={themeColors}
          />
        )}

        {showVideoCall && currentRoom && (
          <VideoCallDialog
            room={currentRoom}
            onClose={() => setShowVideoCall(false)}
            themeColors={themeColors}
          />
        )}

        {showStampPicker && (
          <StampPicker
            onSelect={(stamp, isCustom) => {
              const content = isCustom ? `[スタンプ: ${stamp}]` : stamp;
              sendMessageMutation.mutate({ content, image_url: isCustom ? stamp : null });
            }}
            onClose={() => setShowStampPicker(false)}
            themeColors={themeColors}
          />
        )}

        {showNotificationSettings && (
          <NotificationSettings
            onClose={() => setShowNotificationSettings(false)}
            themeColors={themeColors}
          />
        )}
      </AnimatePresence>

      <Dialog open={showRoomDialog} onOpenChange={setShowRoomDialog}>
        <DialogContent className="w-[90vw] max-w-md">
          <DialogHeader>
            <DialogTitle>{editingRoom ? 'ルームを編集' : '新しいチャットルーム'}</DialogTitle>
          </DialogHeader>
          <div className="space-y-4">
            <div>
              <Label htmlFor="roomName">ルーム名</Label>
              <Input
                id="roomName"
                value={roomFormData.name}
                onChange={(e) => setRoomFormData({ ...roomFormData, name: e.target.value })}
                placeholder="例：プロジェクトA"
                className="mt-2"
              />
            </div>
            <div>
              <Label htmlFor="phoneNumber">電話番号（オプション）</Label>
              <Input
                id="phoneNumber"
                value={roomFormData.phone_number}
                onChange={(e) => setRoomFormData({ ...roomFormData, phone_number: e.target.value })}
                placeholder="例：090-1234-5678"
                className="mt-2"
              />
            </div>
            <div>
              <Label htmlFor="password">入室パスワード（オプション）</Label>
              <Input
                id="password"
                type="password"
                value={roomFormData.password}
                onChange={(e) => setRoomFormData({ ...roomFormData, password: e.target.value })}
                placeholder="パスワードを設定"
                className="mt-2"
              />
            </div>
            <Button
              onClick={() => saveRoomMutation.mutate(roomFormData)}
              disabled={!roomFormData.name.trim()}
              className="w-full"
              style={{
                background: `linear-gradient(135deg, ${themeColors.primary}, ${themeColors.secondary})`
              }}
            >
              {editingRoom ? '更新' : '作成'}
            </Button>
          </div>
        </DialogContent>
      </Dialog>

      <AlertDialog open={!!roomToDelete} onOpenChange={() => setRoomToDelete(null)}>
        <AlertDialogContent>
          <AlertDialogHeader>
            <AlertDialogTitle>ルームを削除しますか？</AlertDialogTitle>
            <AlertDialogDescription>
              「{roomToDelete?.name}」とすべてのメッセージが削除されます。この操作は取り消せません。
            </AlertDialogDescription>
          </AlertDialogHeader>
          <AlertDialogFooter>
            <AlertDialogCancel>キャンセル</AlertDialogCancel>
            <AlertDialogAction
              onClick={() => deleteRoomMutation.mutate(roomToDelete.id)}
              className="bg-red-600 hover:bg-red-700"
            >
              削除
            </AlertDialogAction>
          </AlertDialogFooter>
        </AlertDialogContent>
      </AlertDialog>

      <AlertDialog open={!!messageToDelete} onOpenChange={() => setMessageToDelete(null)}>
        <AlertDialogContent>
          <AlertDialogHeader>
            <AlertDialogTitle>メッセージを削除しますか？</AlertDialogTitle>
            <AlertDialogDescription>
              この操作は取り消せません。
            </AlertDialogDescription>
          </AlertDialogHeader>
          <AlertDialogFooter>
            <AlertDialogCancel>キャンセル</AlertDialogCancel>
            <AlertDialogAction
              onClick={() => deleteMessageMutation.mutate(messageToDelete.id)}
              className="bg-red-600 hover:bg-red-700"
            >
              削除
            </AlertDialogAction>
          </AlertDialogFooter>
        </AlertDialogContent>
      </AlertDialog>

      <Dialog open={showInviteDialog} onOpenChange={setShowInviteDialog}>
        <DialogContent className="w-[90vw] max-w-md bg-white/95 backdrop-blur-2xl border-white/40">
          <DialogHeader>
            <DialogTitle className="flex items-center gap-2">
              <UserPlus className="w-5 h-5" style={{ color: themeColors.primary }} />
              ユーザーを招待
            </DialogTitle>
            <DialogDescription>
              このルームに招待するユーザーのメールアドレスを入力してください
            </DialogDescription>
          </DialogHeader>
          <div className="space-y-4">
            <div>
              <Label htmlFor="inviteEmail">メールアドレス</Label>
              <Input
                id="inviteEmail"
                type="email"
                value={inviteEmail}
                onChange={(e) => setInviteEmail(e.target.value)}
                placeholder="user@example.com"
                className="mt-2"
              />
            </div>
            <Button
              onClick={handleInviteUser}
              disabled={!inviteEmail.trim()}
              className="w-full"
              style={{
                background: `linear-gradient(135deg, ${themeColors.primary}, ${themeColors.secondary})`
              }}
            >
              招待を送信
            </Button>
          </div>
        </DialogContent>
      </Dialog>
    </div>
  );
}
