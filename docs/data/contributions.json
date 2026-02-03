import { mkdir, writeFile } from "fs/promises";

const token = process.env.GITHUB_TOKEN;
if (!token) throw new Error("Missing GITHUB_TOKEN");

const username =
  process.env.USERNAME ||
  process.env.GITHUB_REPOSITORY_OWNER; // default to repo owner

if (!username) throw new Error("Missing USERNAME or GITHUB_REPOSITORY_OWNER");

const repo = process.env.GITHUB_REPOSITORY || "";

const to = new Date();
const from = new Date(to);
from.setFullYear(to.getFullYear() - 1);

const query = `
query($login:String!, $from:DateTime!, $to:DateTime!) {
  user(login: $login) {
    contributionsCollection(from: $from, to: $to) {
      contributionCalendar {
        totalContributions
        weeks {
          firstDay
          contributionDays {
            date
            contributionCount
            color
            weekday
          }
        }
      }
    }
  }
}
`;

const body = {
  query,
  variables: {
    login: username,
    from: from.toISOString(),
    to: to.toISOString(),
  },
};

const res = await fetch("https://api.github.com/graphql", {
  method: "POST",
  headers: {
    "Authorization": `bearer ${token}`,
    "Content-Type": "application/json",
    "User-Agent": "contrib-blaster-sync",
  },
  body: JSON.stringify(body),
});

if (!res.ok) {
  const text = await res.text();
  throw new Error(`GitHub GraphQL failed: ${res.status} ${text}`);
}

const json = await res.json();
if (json.errors?.length) {
  throw new Error(`GraphQL errors: ${JSON.stringify(json.errors)}`);
}

const cal = json.data.user.contributionsCollection.contributionCalendar;
const weeks = cal.weeks || [];

const days = [];
weeks.forEach((w, weekIndex) => {
  (w.contributionDays || []).forEach((d) => {
    days.push({
      ...d,
      weekIndex,
    });
  });
});

const out = {
  generatedAt: new Date().toISOString(),
  username,
  repo,
  from: from.toISOString(),
  to: to.toISOString(),
  total: cal.totalContributions,
  days,
};

await mkdir("docs/data", { recursive: true });
await writeFile("docs/data/contributions.json", JSON.stringify(out, null, 2), "utf-8");

console.log(`Wrote docs/data/contributions.json for ${username} (${days.length} days)`);
