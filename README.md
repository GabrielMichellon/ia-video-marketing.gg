import React, { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Card, CardContent } from "@/components/ui/card";
import { motion } from "framer-motion";
import { initializeApp } from "firebase/app";
import {
  getAuth,
  onAuthStateChanged,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut
} from "firebase/auth";
import {
  getFirestore,
  doc,
  getDoc,
  setDoc,
  updateDoc
} from "firebase/firestore";

const firebaseConfig = {
  apiKey: "AIzaSyChwupVMLYgc3fif7gzr5WQWJcHlC0D8LE",
  authDomain: "ia-video-marketing.firebaseapp.com",
  projectId: "ia-video-marketing",
  storageBucket: "ia-video-marketing.firebasestorage.app",
  messagingSenderId: "125047242705",
  appId: "1:125047242705:web:09d7a6cbee85ba52d52af0"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

export default function IAImageToVideoPage() {
  const [user, setUser] = useState(null);
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [image, setImage] = useState(null);
  const [text, setText] = useState("");
  const [videoUrl, setVideoUrl] = useState("");
  const [loading, setLoading] = useState(false);
  const [energy, setEnergy] = useState(0);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, async (user) => {
      if (user) {
        setUser(user);
        if (user.email === "gabrielmichellon321@gmail.com") {
          setEnergy(Infinity);
        } else {
          const userRef = doc(db, "users", user.uid);
          const docSnap = await getDoc(userRef);
          if (docSnap.exists()) {
            setEnergy(docSnap.data().energy);
          } else {
            await setDoc(userRef, { energy: 10 });
            setEnergy(10);
          }
        }
      } else {
        setUser(null);
      }
    });
    return () => unsubscribe();
  }, []);

  const handleLogin = async () => {
    try {
      await signInWithEmailAndPassword(auth, email, password);
    } catch {
      await createUserWithEmailAndPassword(auth, email, password);
    }
  };

  const handleLogout = async () => {
    await signOut(auth);
  };

  const handleImageUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      const url = URL.createObjectURL(file);
      setImage(url);
    }
  };

  const handleGenerateVideo = async () => {
    if (energy <= 0 && energy !== Infinity) return;
    setLoading(true);
    setTimeout(async () => {
      setVideoUrl("https://www.w3schools.com/html/mov_bbb.mp4");
      setLoading(false);
      if (energy !== Infinity) {
        const userRef = doc(db, "users", user.uid);
        const newEnergy = energy - 1;
        setEnergy(newEnergy);
        await updateDoc(userRef, { energy: newEnergy });
      }
    }, 3000);
  };

  if (!user) {
    return (
      <div className="min-h-screen flex flex-col justify-center items-center bg-gray-900 text-white p-6">
        <h1 className="text-3xl mb-6">Login</h1>
        <Input
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className="mb-4 max-w-sm"
        />
        <Input
          type="password"
          placeholder="Senha"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          className="mb-6 max-w-sm"
        />
        <Button onClick={handleLogin} className="bg-purple-700 hover:bg-purple-800">
          Entrar / Cadastrar
        </Button>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gradient-to-br from-gray-900 via-gray-800 to-purple-900 text-white p-6">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-semibold">Bem-vindo, {user.email}</h1>
        <div className="flex gap-4 items-center">
          <span>Energia: {energy === Infinity ? '∞' : energy}</span>
          <Button onClick={handleLogout} className="bg-red-600 hover:bg-red-700">
            Sair
          </Button>
        </div>
      </div>

      <motion.h1
        className="text-4xl font-bold text-center mb-10"
        initial={{ opacity: 0, y: -20 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6 }}
      >
        Gerador de Vídeos com IA
      </motion.h1>

      <div className="max-w-3xl mx-auto grid gap-6">
        <Card className="bg-gray-800 border border-purple-700">
          <CardContent className="space-y-4 p-6">
            <Input type="file" accept="image/*" onChange={handleImageUpload} />
            {image && <img src={image} alt="preview" className="w-full rounded-xl border border-purple-600" />}

            <Textarea
              placeholder="Digite o texto para a narração do vídeo..."
              value={text}
              onChange={(e) => setText(e.target.value)}
              className="bg-gray-700 border-purple-600"
            />

            <Button
              onClick={handleGenerateVideo}
              className="bg-purple-700 hover:bg-purple-800 text-white w-full"
              disabled={loading || (energy <= 0 && energy !== Infinity)}
            >
              {loading ? "Gerando vídeo..." : "Gerar Vídeo com IA"}
            </Button>

            {energy === 0 && (
              <div className="text-center text-red-400">Sua energia acabou. Compre mais para continuar.</div>
            )}

            {videoUrl && !loading && (
              <video src={videoUrl} controls className="w-full rounded-xl mt-4" />
            )}
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
