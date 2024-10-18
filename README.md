# cuvette
const express = require('express');
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const app = express();

dotenv.config();

// Middleware
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch((err) => console.error(err));

// Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/jobs', require('./routes/jobs'));

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(Server running on port ${PORT}));const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  mobile: { type: String, required: true },
  verified: { type: Boolean, default: false },
});

module.exports = mongoose.model('User', UserSchema);router.post('/login', async (req, res) => {
  const { email, password } = req.body;
  try {
    let user = await User.findOne({ email });
    if (!user) {
      return res.status(400).json({ msg: 'Invalid credentials' });
    }

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
      return res.status(400).json({ msg: 'Invalid credentials' });
    }

    if (!user.verified) {
      return res.status(403).json({ msg: 'Please verify your email first' });
    }

    const payload = { user: { id: user.id } };
    jwt.sign(payload, process.env.JWT_SECRET, { expiresIn: '1h' }, (err, token) => {
      if (err) throw err;
      res.json({ token });
    });
  } catch (err) {
    res.status(500).send('Server error');
  }
});const express = require('express');
const auth = require('../middleware/auth');
const Job = require('../models/Job');
const router = express.Router();

// Create a job post
router.post('/create', auth, async (req, res) => {
  const { title, description, experience, endDate, candidates } = req.body;
  try {
    const newJob = new Job({
      company: req.user.id,
      title,
      description,
      experience,
      endDate,
      candidates,
    });

    const job = await newJob.save();
    res.json(job);
  } catch (err) {
    res.status(500).send('Server error');
  }
});

module.exports = router;const nodemailer = require('nodemailer');

router.post('/create', auth, async (req, res) => {
  const { title, description, experience, endDate, candidates } = req.body;
  try {
    const newJob = new Job({
      company: req.user.id,
      title,
      description,
      experience,
      endDate,
      candidates,
    });

    const job = await newJob.save();

    // Send email notifications to candidates
    const transporter = nodemailer.createTransport({
      service: 'Gmail',
      auth: {
        user: process.env.EMAIL_USER,
        pass: process.env.EMAIL_PASS,
      },
    });

    candidates.forEach(candidate => {
      const mailOptions = {
        from: process.env.EMAIL_USER,
        to: candidate.email,
        subject: Job Alert: ${title},
        text: New Job: ${title}\nDescription: ${description}\nExperience Required: ${experience},
      };

      transporter.sendMail(mailOptions, (error, info) => {
        if (error) {
          console.error(Error sending email to ${candidate.email});
        } else {
          console.log(Email sent to ${candidate.email});
        }
      });
    });

    res.json(job);
  } catch (err) {
    res.status(500).send('Server error');
  }
});
