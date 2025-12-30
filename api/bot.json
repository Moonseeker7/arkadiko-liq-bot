require('dotenv').config();
const { Telegraf } = require('telegraf');
const { webhookCallback } = require('telegraf');
const { callReadOnlyFunction, uintCV } = require('@stacks/transactions');
const { StacksMainnet } = require('@stacks/network');

const bot = new Telegraf(process.env.BOT_TOKEN);
const network = new StacksMainnet();

const VAULTS_CONTRACT = 'SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR';
const LIQ_THRESHOLD = 150;

bot.start((ctx) => ctx.reply('Welcome to Arkadiko Liquidation Checker! 🚀\nUse /check <vault-id>\nExample: /check 123'));

bot.help((ctx) => ctx.reply('Commands:\n/start - welcome\n/check <number> - check vault ratio'));

bot.command('check', async (ctx) => {
  const args = ctx.message.text.split(' ');
  if (args.length < 2) return ctx.reply('Add vault ID! Example: /check 123');

  const vaultId = args[1];
  if (isNaN(vaultId)) return ctx.reply('Vault ID must be a number!');

  try {
    ctx.reply(`🔍 Checking vault #${vaultId}...`);

    const ratioResponse = await callReadOnlyFunction({
      contractAddress: VAULTS_CONTRACT,
      contractName: 'arkadiko-vaults-data-v1-1',  // ← confirm this name on hiro.so if error
      functionName: 'get-collateral-ratio-for-vault',
      functionArgs: [uintCV(Number(vaultId))],
      senderAddress: 'SP00000000000000000000000000000000ZZZZZZ', // dummy
      network
    });

    const ratioScaled = ratioResponse.value.value;
    const ratio = ratioScaled / 10000;

    let msg = `Vault #\( {vaultId}\nCollateral Ratio: ** \){ratio.toFixed(2)}%**\n\n`;

    if (ratio < LIQ_THRESHOLD) {
      msg += '⚠️ DANGER! Liquidatable now! Opportunity!';
    } else if (ratio < 170) {
      msg += '🔴 Warning: Approaching liquidation';
    } else {
      msg += '✅ Safe';
    }

    ctx.replyWithMarkdown(msg);
  } catch (error) {
    console.error(error);
    ctx.reply('Error 😔 Wrong ID or network issue?');
  }
});

// Export for Vercel webhook
module.exports = webhookCallback(bot, 'express');
