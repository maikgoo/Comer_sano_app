const { createClient } = require('redis');

let client = null;

async function getClient() {
  if (!client) {
    client = createClient({ url: process.env.REDIS_URL });
    client.on('error', (err) => console.error('Redis error:', err));
    await client.connect();
  }
  return client;
}

module.exports = async function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, DELETE, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');
  if (req.method === 'OPTIONS') return res.status(200).end();

  try {
    const r = await getClient();
    const { action, hogarId, profileId, data } = req.method === 'POST'
      ? req.body
      : req.query;

    if (!hogarId) return res.status(400).json({ error: 'hogarId required' });

    // GET perfiles del hogar
    if (action === 'getProfiles') {
      const raw = await r.get(`hogar:${hogarId}:profiles`);
      return res.json({ profiles: raw ? JSON.parse(raw) : [] });
    }

    // SAVE perfiles
    if (action === 'saveProfiles') {
      await r.set(`hogar:${hogarId}:profiles`, JSON.stringify(data));
      return res.json({ ok: true });
    }

    // GET recetas guardadas de un perfil
    if (action === 'getSaved') {
      if (!profileId) return res.status(400).json({ error: 'profileId required' });
      const raw = await r.get(`hogar:${hogarId}:profile:${profileId}:saved`);
      return res.json({ saved: raw ? JSON.parse(raw) : [] });
    }

    // SAVE receta
    if (action === 'saveRecipe') {
      if (!profileId) return res.status(400).json({ error: 'profileId required' });
      const key = `hogar:${hogarId}:profile:${profileId}:saved`;
      const raw = await r.get(key);
      const saved = raw ? JSON.parse(raw) : [];
      // Evitar duplicados por nombre
      const exists = saved.find(s => s.name === data.name);
      if (!exists) saved.unshift({ ...data, savedAt: Date.now() });
      await r.set(key, JSON.stringify(saved));
      return res.json({ ok: true, count: saved.length });
    }

    // DELETE receta
    if (action === 'deleteRecipe') {
      if (!profileId) return res.status(400).json({ error: 'profileId required' });
      const key = `hogar:${hogarId}:profile:${profileId}:saved`;
      const raw = await r.get(key);
      const saved = raw ? JSON.parse(raw) : [];
      const filtered = saved.filter(s => s.savedAt !== parseInt(data.savedAt));
      await r.set(key, JSON.stringify(filtered));
      return res.json({ ok: true, count: filtered.length });
    }

    return res.status(400).json({ error: 'Unknown action' });

  } catch (err) {
    console.error(err);
    return res.status(500).json({ error: err.message });
  }
};

module.exports.config = { maxDuration: 10 };
