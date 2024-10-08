<template>
  <div v-if="loading" class="loading-container">
    <div ref="lottieAnimation" class="lottie-animation"></div>
  </div>

  <div class="league-container" v-if="!loading">
    <div class="layout-grid">
      <div class="left-section">
        <div class="league-results-card">
          <div class="league-table-wrapper">
            <h2>League Table</h2>
            <table class="league-table">
              <thead>
                <tr>
                  <th>Teams</th>
                  <th>Played</th>
                  <th>Won</th>
                  <th>Drawn</th>
                  <th>Lost</th>
                  <th>GF</th>
                  <th>GA</th>
                  <th>GD</th>
                  <th>Points</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="team in leagueTable" :key="team.id">
                  <td>{{ team.name }}</td>
                  <td>{{ team.matches_played }}</td>
                  <td>{{ team.matches_won }}</td>
                  <td>{{ team.matches_drawn }}</td>
                  <td>{{ team.matches_lost }}</td>
                  <td>{{ team.goals_for }}</td>
                  <td>{{ team.goals_against }}</td>
                  <td>{{ team.goal_difference }}</td>
                  <td>{{ team.points }}</td>
                </tr>
              </tbody>
            </table>
          </div>
          <div class="match-results-wrapper">
            <h2>Match Results</h2>
            <p>{{ getOrdinal(currentWeek) }} Week Match Result</p>
            <table>
              <tbody>
                <tr v-for="match in matchResults" :key="match.id">
                  <td>{{ match.home_team }}</td>
                  <td>
                    <span v-if="currentEditingId !== match.id"
                      >{{ match.home_team_score }} -
                      {{ match.away_team_score }}</span
                    >
                    <div v-else>
                      <input
                        type="number"
                        v-model="match.home_team_score"
                        class="edit-input"
                      />
                      -
                      <input
                        type="number"
                        v-model="match.away_team_score"
                        class="edit-input"
                      />
                    </div>
                  </td>
                  <td>{{ match.away_team }}</td>
                  <td>
                    <span @click="toggleEdit(match.id)">
                      <img
                        :src="
                          currentEditingId === match.id ? saveIcon : editIcon
                        "
                        alt="Edit/Save Icon"
                        class="icon"
                      />
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
        <div class="button-actions">
          <button @click="playAllMatches">Play All</button>
          <button @click="resetMatches" class="reset-button">Reset All</button>
          <button @click="nextWeekMatches">Next Week</button>
        </div>
      </div>
      <div class="right-section">
        <div class="predictions-card">
          <h2>
            {{ getOrdinal(currentWeek) }} Week Predictions of Championship
          </h2>
          <table class="predictions-table">
            <tbody>
              <tr v-for="(prediction, index) in predictions" :key="index">
                <td>{{ prediction.name }}</td>
                <td>{{ prediction.win_probability }}%</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import lottie from "lottie-web";

interface Team {
  id: number;
  name: string;
  matches_played: number;
  matches_won: number;
  matches_drawn: number;
  matches_lost: number;
  goals_for: number;
  goals_against: number;
  goal_difference: number;
  points: number;
}

interface MatchResult {
  home_team: string;
  away_team: string;
  home_team_score: number;
  away_team_score: number;
  id: number;
}

interface Prediction {
  name: string;
  win_probability: number;
}
const editIcon: string =
  "https://img.icons8.com/material-outlined/24/000000/edit.png";
const saveIcon: string =
  "https://img.icons8.com/material-outlined/24/000000/save.png";

const leagueTable = ref<Team[]>([]);
const matchResults = ref<MatchResult[]>([]);
const predictions = ref<Prediction[]>([]);

const currentEditingId = ref<number | null>(null);
const loading = ref<boolean>(false);
const allMatches = ref<boolean>(false);
const currentWeek = ref<number>(0);
const totalWeek = ref<number>(0);
const baseUrl = "http://localhost:8000/api";
const lottieAnimation = ref<HTMLDivElement | null>(null);
const fetchData = async () => {
  loading.value = true;
  try {
    const [leagueRes, matchRes, predictionRes] = await Promise.all([
      fetch(baseUrl + "/teams").then((res) => res.json()),
      fetch(
        baseUrl + `${allMatches.value ? "/matches" : "/matches/last-week"}`
      ).then((res) => res.json()),
      fetch(baseUrl + "/league/predictions").then((res) => res.json()),
    ]);
    leagueTable.value = leagueRes;
    matchResults.value = matchRes;
    predictions.value = predictionRes;
    totalWeek.value = leagueRes.length - 1;
    if (matchRes && matchRes.length > 0) {
      currentWeek.value = matchRes[0].week;
    }
  } catch (error) {
    console.error("Error fetching data:", error);
  } finally {
    setTimeout(() => {
      loading.value = false;
    }, 1000);
  }
};

onMounted(async () => {
  await fetchData();

  if (lottieAnimation.value) {
    lottie.loadAnimation({
      container: lottieAnimation.value,
      renderer: "svg",
      loop: true,
      autoplay: true,
      path: "/src/assets/loading-animation.json",
    });
  }
});

onBeforeUnmount(() => {
  if (lottieAnimation.value) {
    lottie.destroy();
  }
});

const toggleEdit = async (id: number) => {
  if (currentEditingId.value === id) {
    const match = matchResults.value.find((m) => m.id === id);
    if (match) {
      console.log(match.id);
      try {
        const response = await fetch(baseUrl + "/matches/" + match.id, {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            home_team_score: match.home_team_score,
            away_team_score: match.away_team_score,
          }),
        });
        if (response.ok) {
          fetchData();
        } else {
          alert("Failed to update match result:");
        }
      } catch (error) {
        alert("Failed to update match result:");
      }
    }
    currentEditingId.value = null;
  } else {
    currentEditingId.value = id;
  }
};

const playAllMatches = async () => {
  try {
    allMatches.value = true;
    const response = await fetch(baseUrl + "/league/simulate-all", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({}),
    });
    if (response.ok) {
      fetchData();
    } else {
      alert("Failed to play All");
    }
  } catch (error) {
    alert("Failed to play All");
  }
};

const resetMatches = async () => {
  try {
    allMatches.value = false;
    const response = await fetch(baseUrl + "/league/reset", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({}),
    });
    if (response.ok) {
      fetchData();
    } else {
      alert("Failed to reset All");
    }
  } catch (error) {
    alert("Failed to reset All");
  }
};

const nextWeekMatches = async () => {
  try {
    if (currentWeek.value + 1 == totalWeek.value) {
      allMatches.value = true;
    }
    const response = await fetch(baseUrl + "/league/simulate-week", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({}),
    });
    if (response.ok) {
      fetchData();
    } else {
      alert("Failed to play week");
    }
  } catch (error) {
    alert("Failed to play week");
  }
};

const getOrdinal = (n: number) => {
  const s = ["th", "st", "nd", "rd"];
  const v = n % 100;
  return n + (s[(v - 20) % 10] || s[v] || s[0]);
};
</script>

<style scoped lang="scss">
@import "../styles/league-table.scss";
</style>
