import React, { useState, useEffect, useRef } from "react";
import { SafeAreaView, View, Text, StyleSheet, TouchableOpacity, TextInput, FlatList, ScrollView, Alert, StatusBar } from "react-native";
import Constants from "expo-constants";
import { StatusBar as ExpoStatusBar } from "expo-status-bar";
import AsyncStorage from "@react-native-async-storage/async-storage";
import { Audio } from "expo-av";

// === Main App ===
export default function App() {
  const [view, setView] = useState("home"); // home | tasks | stats
  const [tasks, setTasks] = useState([]);
  const [sessions, setSessions] = useState([]);

  useEffect(() => {
    (async () => {
      const t = await AsyncStorage.getItem("@msr_tasks");
      const s = await AsyncStorage.getItem("@msr_sessions");
      if (t) setTasks(JSON.parse(t));
      if (s) setSessions(JSON.parse(s));
    })();
  }, []);

  useEffect(() => {
    AsyncStorage.setItem("@msr_tasks", JSON.stringify(tasks));
  }, [tasks]);

  useEffect(() => {
    AsyncStorage.setItem("@msr_sessions", JSON.stringify(sessions));
  }, [sessions]);

  const addSession = (minutes) => {
    const today = new Date().toISOString().slice(0, 10);
    setSessions((prev) => {
      const copy = [...prev];
      const idx = copy.findIndex((p) => p.date === today);
      if (idx >= 0) copy[idx].minutes += minutes;
      else copy.push({ date: today, minutes });
      return copy;
    });
  };

  return (
    <SafeAreaView style={styles.container}>
      <StatusBar hidden />
      <View style={styles.header}>
        <Text style={styles.title}>My Study Room</Text>
        <View style={styles.nav}>
          <TouchableOpacity onPress={() => setView("home")} style={styles.navBtn}>
            <Text style={styles.navTxt}>خانه</Text>
          </TouchableOpacity>
          <TouchableOpacity onPress={() => setView("tasks")} style={styles.navBtn}>
            <Text style={styles.navTxt}>تسک‌ها</Text>
          </TouchableOpacity>
          <TouchableOpacity onPress={() => setView("stats")} style={styles.navBtn}>
            <Text style={styles.navTxt}>آمار</Text>
          </TouchableOpacity>
        </View>
      </View>

      <View style={styles.body}>
        {view === "home" && (
          <>
            <Timer onSessionComplete={(minutes) => addSession(minutes)} tasks={tasks} />
            <View style={{ marginTop: 12 }}>
              <Text style={{ textAlign: "center", color: "#666" }}>
                جلسات امروز:{" "}
                {sessions.find((s) => s.date === new Date().toISOString().slice(0, 10))?.minutes || 0} دقیقه
              </Text>
            </View>
          </>
        )}
        {view === "tasks" && <Tasks tasks={tasks} setTasks={setTasks} />}
        {view === "stats" && <Stats sessions={sessions} />}
      </View>

      <ExpoStatusBar style="auto" />
    </SafeAreaView>
  );
}

// === Timer Component ===
function Timer({ onSessionComplete, tasks }) {
  const [minutes, setMinutes] = useState("25");
  const [running, setRunning] = useState(false);
  const [remaining, setRemaining] = useState(0);
  const intervalRef = useRef(null);
  const soundRef = useRef(null);

  useEffect(() => {
    return () => {
      if (intervalRef.current) clearInterval(intervalRef.current);
      if (soundRef.current) soundRef.current.unloadAsync();
    };
  }, []);

  useEffect(() => {
    if (!running) return;
    intervalRef.current = setInterval(() => {
      setRemaining((r) => {
        if (r <= 1) {
          clearInterval(intervalRef.current);
          setRunning(false);
          playSound();
          onSessionComplete(parseInt(minutes || "0"));
          return 0;
        }
        return r - 1;
      });
    }, 1000);
    return () => clearInterval(intervalRef.current);
  }, [running]);

  const start = () => {
    const m = parseInt(minutes || "0");
    if (!m || m <= 0) return Alert.alert("مدت نامعتبر", "لطفاً دقیقه درست وارد کن.");
    setRemaining(m * 60);
    setRunning(true);
  };

  const stop = () => {
    setRunning(false);
    if (intervalRef.current) clearInterval(intervalRef.current);
  };

  const playSound = async () => {
    try {
      const { sound } = await Audio.Sound.createAsync(require("./assets/calm.mp3"));
      soundRef.current = sound;
      await sound.playAsync();
    } catch (e) {
      // اگه فایل نبود نادیده بگیر
    }
  };

  const formatTime = (s) => {
    const mm = Math.floor(s / 60).toString().padStart(2, "0");
    const ss = (s % 60).toString().padStart(2, "0");
    return `${mm}:${ss}`;
  };

  return (
    <View style={card.card}>
      <Text style={card.heading}>تایمر مطالعه</Text>
      <View style={{ flexDirection: "row", justifyContent: "center", alignItems: "center" }}>
        <TextInput
          keyboardType="numeric"
          value={minutes}
          onChangeText={setMinutes}
          style={card.input}
          placeholder="دقیقه"
        />
        <TouchableOpacity onPress={start} style={card.btn}>
          <Text style={card.btnTxt}>شروع</Text>
        </TouchableOpacity>
        <TouchableOpacity onPress={stop} style={[card.btn, { backgroundColor: "#bbb" }]}>
          <Text style={card.btnTxt}>توقف</Text>
        </TouchableOpacity>
      </View>

      <View style={{ marginTop: 16, alignItems: "center" }}>
        <Text style={card.time}>
          {running ? formatTime(remaining) : `${minutes.padStart(2, "0")}:00`}
        </Text>
      </View>

      <View style={{ marginTop: 14 }}>
        <Text style={{ fontWeight: "600" }}>تسک‌های اخیر</Text>
        {tasks.length === 0 ? (
          <Text style={{ color: "#666", marginTop: 6 }}>هنوز تسکی وجود نداره</Text>
        ) : (
          tasks.slice(0, 3).map((t, i) => (
            <Text key={i} style={{ marginTop: 6 }}>
              • {t.title} {t.done ? "(تمام شده)" : ""}
            </Text>
          ))
        )}
      </View>
    </View>
  );
}

// === Tasks Component ===
function Tasks({ tasks, setTasks }) {
  const [title, setTitle] = useState("");

  const add = () => {
    if (!title.trim()) return Alert.alert("خالی است", "عنوان تسک را وارد کنید.");
    setTasks((prev) => [{ id: Date.now().toString(), title: title.trim(), done: false }, ...prev]);
    setTitle("");
  };

  const toggle = (id) => {
    setTasks((prev) => prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t)));
  };

  const removeDone = () => {
    setTasks((prev) => prev.filter((t) => !t.done));
  };

  return (
    <View>
      <Text style={card.heading}>تسک‌ها</Text>
      <View style={{ flexDirection: "row", marginTop: 8 }}>
        <TextInput value={title} onChangeText={setTitle} placeholder="عنوان تسک..." style={card.inputFlex} />
        <TouchableOpacity onPress={add} style={card.addBtn}>
          <Text style={{ color: "#fff" }}>اضافه</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        style={{ marginTop: 12 }}
        data={tasks}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TouchableOpacity onPress={() => toggle(item.id)} style={card.taskRow}>
            <Text style={{ textDecorationLine: item.done ? "line-through" : "none" }}>{item.title}</Text>
            <Text>{item.done ? "✅" : "⬜"}</Text>
          </TouchableOpacity>
        )}
        ListEmptyComponent={<Text style={{ color: "#666", marginTop: 10 }}>هیچ تسکی اضافه نشده</Text>}
      />

      <TouchableOpacity onPress={removeDone} style={{ marginTop: 12, alignSelf: "center", padding: 8 }}>
        <Text style={{ color: "#d00" }}>حذف تمام‌شده‌ها</Text>
      </TouchableOpacity>
    </View>
  );
}

// === Stats Component ===
function Stats({ sessions }) {
  const last7 = [];
  for (let i = 6; i >= 0; i--) {
    const d = new Date();
    d.setDate(d.getDate() - i);
    const dateStr = d.toISOString().slice(0, 10);
    const s = sessions.find((x) => x.date === dateStr);
    last7.push({ date: dateStr, minutes: s ? s.minutes : 0 });
  }
  const total = last7.reduce((a, b) => a + b.minutes, 0);

  return (
    <ScrollView>
      <Text style={card.heading}>آمار هفت روز اخیر</Text>
      <View style={{ marginTop: 12 }}>
        {last7.map((d, i) => (
          <View key={i} style={card.statRow}>
            <Text style={{ width: 100 }}>{d.date.slice(5)}</Text>
            <View style={card.barWrapper}>
              <View style={[card.bar, { width: Math.min(300, d.minutes * 3) }]} />
            </View>
            <Text style={{ width: 60, textAlign: "right" }}>{d.minutes} min</Text>
          </View>
        ))}
      </View>
      <View style={{ marginTop: 16 }}>
        <Text style={{ fontWeight: "700" }}>کل دقیقه در ۷ روز: {total} دقیقه</Text>
      </View>
    </ScrollView>
  );
}

// === Styles ===
const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: Constants.statusBarHeight, backgroundColor: "#f7f7f8" },
  header: { padding: 16, borderBottomWidth: 1, borderColor: "#eee" },
  title: { fontSize: 18, fontWeight: "700", textAlign: "center" },
  nav: { flexDirection: "row", justifyContent: "space-around", marginTop: 12 },
  navBtn: { padding: 8 },
  navTxt: { color: "#007aff", fontWeight: "600" },
  body: { flex: 1, padding: 16 }
});

const card = StyleSheet.create({
  card: { backgroundColor: "#fff", padding: 14, borderRadius: 10, shadowColor: "#00000011", elevation: 2 },
  heading: { fontSize: 16, fontWeight: "700" },
  input: { borderWidth: 1, borderColor: "#ddd", padding: 8, width: 80, marginRight: 8, borderRadius: 6, textAlign: "center" },
  inputFlex: { flex: 1, borderWidth: 1, borderColor: "#ddd", borderRadius: 6, padding: 8 },
  btn: { backgroundColor: "#007aff", padding: 10, borderRadius: 6, marginLeft: 8 },
  btnTxt: { color: "#fff", fontWeight: "700" },
  time: { fontSize: 36, fontWeight: "800" },
  addBtn: { marginLeft: 8, backgroundColor: "#28a745", padding: 10, borderRadius: 6 },
  taskRow: { flexDirection: "row", justifyContent: "space-between", padding: 10, backgroundColor: "#fff", marginBottom: 8, borderRadius: 8 },
  statRow: { flexDirection: "row", alignItems: "center", marginBottom: 10 },
  barWrapper: { flex: 1, height: 14, backgroundColor: "#eee", borderRadius: 8, marginHorizontal: 8 },
  bar: { height: 14, backgroundColor: "#007aff", borderRadius: 8 }
});
