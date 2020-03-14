using Harmony12;
using System;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;

namespace DreamLover
{
	[HarmonyPatch(typeof(PeopleLifeAI), "UpdateTileCharsBehavior")]
	public static class AutoRape_Patch
	{
		public static PatchModuleInfo patchModuleInfo = new PatchModuleInfo(
			typeof(PeopleLifeAI), "UpdateTileCharsBehavior",
			typeof(AutoRape_Patch));
		private static void Debug(string str)
		{
			Main.Debug("<主动欺辱> " + str);
		}
		public static bool Prefix(int mapId, int tileId, bool isTaiwuAtThisTile, Dictionary<int, int> righteousInfo, object disasterInfo, int worldId, int mainActorId, Dictionary<int, List<int>> mainActorItems, System.Random random)
		{
			if (!Main.enabled || !Main.settings.rape.autorape.Enabled)
			{
				return true;
			}

			if (!isTaiwuAtThisTile)
			{
				return true;
			}

			Debug("开始寻找目标");

			int 角色立场 = DateFile.instance.GetActorGoodness(mainActorId);
			int 欺辱概率 = int.Parse(DateFile.instance.goodnessDate[角色立场][25]);
			int 战力评价 = int.Parse(DateFile.instance.GetActorDate(mainActorId, 993, applyBonus: false));
			int 性别 = int.Parse(DateFile.instance.GetActorDate(mainActorId, 14, applyBonus: false));

			PeopleLifeAIHelper.GetTileCharacters(mapId, tileId, out int[] aliveChars);
			List<int> list = aliveChars.ToList();

			if (Main.settings.rape.autorape.JustLover)
			{
				list = list.Where((int id) => DateFile.instance.GetActorSocial(mainActorId, 312).Contains(id)).ToList();
			}
			if (Main.settings.rape.autorape.FilterName)
			{
				try
				{
					list = list.Where((int id) => DateFile.instance.GetActorName(id).IndexOf(Main.settings.rape.autorape.Name) != -1).ToList();
				}
				catch (Exception e)
				{
					Debug("地块有角色姓名获取失败，无法使用姓名过滤，主动欺辱判定强行终止。");
					Debug(e.ToString());
					return true;
				}
			}
			if (Main.settings.rape.autorape.DifferentSex)
			{
				list = list.Where((int id) => int.Parse(DateFile.instance.GetActorDate(id, 14, applyBonus: false)) != 性别).ToList();
			}

			string names = "";
			foreach(int kid in list)
			{
				try
				{
					names += DateFile.instance.GetActorName(kid) + " ";
				}
				catch (Exception)
				{
					list.Remove(kid);
					Debug(string.Format("{0} 无法获取姓名，将从列表中移除", kid));
				}
			}

			if(list.Count == 0)
