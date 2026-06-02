---
layout: page
title: SWE Notes
---

<div class="card-grid">
  <a href="fastapi" class="card">
    <h3>Backend & APIs</h3>
    <p>FastAPI, SQLAlchemy, SQLite, asyncio, pytest</p>
  </a>
  <a href="openai-streaming" class="card">
    <h3>LLM & AI</h3>
    <p>OpenAI Streaming, OpenAI Tools, LangChain</p>
  </a>
  <a href="aws-lambda" class="card">
    <h3>AWS</h3>
    <p>Lambda, DynamoDB, EventBridge</p>
  </a>
  <a href="terraform" class="card">
    <h3>Infrastructure</h3>
    <p>Terraform, Docker</p>
  </a>
</div>

<style>
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  margin-top: 2rem;
}
.card {
  background: #2a2a3e;
  border-radius: 8px;
  padding: 1.5rem;
  text-decoration: none;
  transition: transform 0.2s, box-shadow 0.2s;
}
.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
}
.card h3 {
  color: #6db3f2;
  margin-top: 0;
}
.card p {
  color: #b0b0b0;
  margin-bottom: 0;
}
</style>
